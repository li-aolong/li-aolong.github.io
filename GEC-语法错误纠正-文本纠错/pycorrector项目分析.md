# 基于深度模型的方法

## bert

### predict_mask.py

预测一个句子中被 mask 掉的字，提供概率值top5的候选字

### config.py

使用中文语料fine-tune过的模型位置，字典位置，输出位置，句子最大长度，阈值（默认0.1）

### bert_detector.py

首先使用`initialize_bert_detector()`进行初始化，加载BertTokenizer，[MASK]字符串及对应ID，bert模型

```python
def initialize_bert_detector(self):
    t1 = time.time()
    self.bert_tokenizer = BertTokenizer(vocab_file=self.bert_model_vocab)
    self.MASK_TOKEN = "[MASK]"
    self.MASK_ID = self.bert_tokenizer.convert_tokens_to_ids([self.MASK_TOKEN])[0]
    # Prepare model
    self.model = BertForMaskedLM.from_pretrained(self.bert_model_dir)
    logger.debug("Loaded model ok, path: %s, spend: %.3f s." % (self.bert_model_dir, time.time() - t1))
    self.initialized_bert_detector = True
```

然后使用`_convert_sentence_to_detect_features()`将句子中的每个token转化为一个feature，该feature包含了：

- input_tokens（句子被拆分为的一串tokens，都一样）

- input_ids（tokens对应的id，都一样）
- masked_lm_labels（掩码处为token_id，其余位置为-1）
- id（该token在句子中的序号）
- token（该touken）

```python
eval_features = self._convert_sentence_to_detect_features(sentence)
```

然后使用`predict_token_prob()`对句子中的每个feature进行计算loss，model的参数中加入了masked_lm_labels，则是计算对应mask位置的loss，根据loss计算该位置的概率值

```python
def predict_token_prob(self, sentence):
    self.check_bert_detector_initialized()
    result = []
    eval_features = self._convert_sentence_to_detect_features(sentence)

    for f in eval_features:
        input_ids = torch.tensor([f.input_ids])
        masked_lm_labels = torch.tensor([f.masked_lm_labels])
        outputs = self.model(input_ids, masked_lm_labels=masked_lm_labels)
        # 检测阶段用loss，没用predictions
        masked_lm_loss, predictions = outputs[:2]
        prob = np.exp(-masked_lm_loss.item())
        result.append([prob, f])
    return result
```

使用`detect()`来检测句子`sentence`中可能的错误，若句子中token的概率值该值小于阈值`threshold`（默认0.1），则记录为可能出错

```python
def detect(self, sentence):
    """
    句子改错
    :param sentence: 句子文本
    :param threshold: 阈值
    :return: list[list], [error_word, begin_pos, end_pos, error_type]
    """
    maybe_errors = []
    for prob, f in self.predict_token_prob(sentence):
        # 日志信息
        logger.debug('token:%s, idx:%s, prob:%s' % (f.token, f.id, prob))
        if prob < self.threshold:
            maybe_errors.append([f.token, f.id, f.id + 1, ErrorType.char])
    return maybe_errors
```

### bert_corrector.py

首先使用detect()对句子进行检错，对句子中可能出错的token进行逐个纠正。

该程序不处理非中文，使用`_convert_sentence_to_correct_features()`来将句子中可能出错的token转换为一个feature，该feature包含了：

- input_tokens（加上标志符的一串tokens，其中可能出错的token被mask，没有padding）

- input_ids（tokens对应的id，其中可能出错的token被mask）
- mask_ids（被mask位置的id，如：[1]）
- segment_ids（句子标志，128长度的0穿，都一样）

```python
def _convert_sentence_to_correct_features(self, sentence, error_begin_idx, error_end_idx):
    """Loads a sentence into a list of `InputBatch`s."""
    features = []
    tokens_a = self.bert_tokenizer.tokenize(sentence)

    # For single sequences:
    #  tokens:   [CLS] the dog is hairy . [SEP]
    #  type_ids: 0      0   0   0  0    0   0
    tokens = ["[CLS]"] + tokens_a + ["[SEP]"]
    k = error_begin_idx + 1
    for i in range(error_end_idx - error_begin_idx):
        tokens[k] = '[MASK]'
        k += 1
    segment_ids = [0] * len(tokens)

    input_ids = self.bert_tokenizer.convert_tokens_to_ids(tokens)
    mask_ids = [i for i, v in enumerate(input_ids) if v == self.MASK_ID]

    # Zero-pad up to the sequence length. 
    padding = [0] * (self.max_seq_length - len(input_ids))
    input_ids += padding
    segment_ids += padding

    features.append(
        InputFeatures(input_ids=input_ids,
                      mask_ids=mask_ids,
                      segment_ids=segment_ids,
                      input_tokens=tokens))

    return features
```

然后使用model进行计算，model参数中加入了token_type_ids，输出为每个token在字典中的分数，然后找出分数最高的index即为程序预测最可能的序号predicted_index，再转化成对应的token，就是程序预测的纠正corrected_item

```python
def predict_mask_token(self, sentence, error_begin_idx, error_end_idx):
    corrected_item = sentence[error_begin_idx:error_end_idx]
    eval_features = self._convert_sentence_to_correct_features(
        sentence=sentence,
        error_begin_idx=error_begin_idx,
        error_end_idx=error_end_idx
    )
    assert len(eval_features) == 1

    for f in eval_features:
        input_ids = torch.tensor([f.input_ids])
        segment_ids = torch.tensor([f.segment_ids])
        outputs = self.model(input_ids, token_type_ids=segment_ids)
        # 纠错阶段使用predictions：scores for each vocabulary token before SoftMax
        predictions = outputs[0]    # predictions的shape为：[1, 128, 21128]
        
        # confirm we were able to predict 'henson'
        masked_ids = f.mask_ids
        
        assert len(masked_ids) == 1
        if masked_ids:
            for idx, i in enumerate(masked_ids):
                predicted_index = torch.argmax(predictions[0, i]).item()
                predicted_token = self.bert_tokenizer.convert_ids_to_tokens([predicted_index])[0]
                # 日志信息
#                     logger.debug('original text is: %s' % f.input_tokens)
#                     logger.debug('Mask predict is: %s' % predicted_token)
                corrected_item = predicted_token
    return corrected_item
```

如果预测值与原值不一致，则使用预测值输出纠正后的句子