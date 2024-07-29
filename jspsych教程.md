这是一篇详细的教程，旨在帮助那些不熟悉 JsPsych 的人设计并运行一个心理学实验。我们将以的 n-back 任务为例，逐步解释如何使用 JsPsych 设计实验。

### 1. 了解 JsPsych

**JsPsych** 是一个用于在网页上创建心理学实验的 JavaScript 库。它允许研究人员轻松地创建各种类型的实验，如记忆测试、反应时间测试等。JsPsych 的主要优势在于它的灵活性和易用性。

### 2. 准备实验环境

首先，我们需要一个可以编写和运行 HTML 文件的环境。我们可以使用任何文本编辑器（如 Notepad++、Sublime Text 或 Visual Studio Code）来编写代码，并在任何现代浏览器（如 Chrome、Firefox 或 Safari）中运行它。

### 3. 创建 HTML 文件

打开文本编辑器，创建一个新的 HTML 文件，并输入以下基本结构：

```html
<!DOCTYPE html>
<html>
  <head>
    <title>JsPsych Experiment</title>
  </head>
  <body>
    <script src="jspsych-6.1.0/jspsych.js"></script>
    <script src="jspsych-6.1.0/plugins/jspsych-html-keyboard-response.js"></script>
    <script src="jspsych-6.1.0/plugins/jspsych-html-button-response.js"></script>
    <link rel="stylesheet" href="jspsych-6.1.0/css/jspsych.css">
  </body>
</html>
```

**说明：**
- `<script>` 标签用于引入 JsPsych 库及其插件。
- `<link>` 标签用于引入 JsPsych 的样式表。

### 4. 初始化实验变量

在 `<body>` 标签的 `<script>` 标签内，定义实验所需的变量：

```javascript
<script>
  var timeline = [];

  var n_back_set = ['Z', 'X', 'C', 'V', 'B', 'N'];
  var sequence = [];

  var how_many_back = 2;
  var sequence_length = 22;
</script>
```

**说明：**
- `timeline`：存储实验流程的数组。
- `n_back_set`：用于 n-back 任务的字母集合。
- `sequence`：存储生成的字母序列的数组。
- `how_many_back`：定义了 n-back 任务的难度（这里是 2-back）。
- `sequence_length`：定义了任务的总长度（这里是 22 个字母）。

### 5. 添加实验指导语

在 `timeline` 数组中添加指导语步骤：

```javascript
var instructions_1 = {
  type: 'html-button-response',
  stimulus: '<div style="width: 800px;">'+ 
    '<p>This experiment will test your ability to hold information in short-term, temporary memory. This is called working memory.</p>'+
    '</div>',
  choices: ["Continue"]
}
timeline.push(instructions_1);

var instructions_2 = {
  type: 'html-button-response',
  stimulus: '<div style="width: 800px;">'+ 
    '<p>You will see a sequence of letters presented one at a time. Your task is to determine if the letter on the screen matches '+
    'the letter that appeared two letters before.</p>'+
    '<p>If the letter is match <span style="font-weight: bold;">press the M key.</span></p>'+
    '<p>For example, if you saw the sequence X, C, V, B, V, X you would press the M key when the second V appeared on the screen.</p>'+
    '<p>You do not need to press any key when there is not a match.</p>'+
    '</div>',
  choices: ["Continue"]
}
timeline.push(instructions_2);

var instructions_3 = {
  type: 'html-button-response',
  stimulus: '<div style="width: 800px;">'+ 
    '<p>The sequence will begin on the next screen.</p>'+
    '<p>Remember: press the M key if the letter on the screen matches the letter that appeared two letters ago.</p>'+
    '</div>',
  choices: ["I'm ready to start!"],
  post_trial_gap: 1000
}
timeline.push(instructions_3);
```

**说明：**
- `type: 'html-button-response'`：指定这是一个按钮响应类型的试验。
- `stimulus`：定义显示给参与者的文本。
- `choices`：定义参与者可以选择的按钮。
- `post_trial_gap`：定义试验结束后的间隔时间。

### 6. 定义 n-back 任务试验

在 `timeline` 数组中添加 n-back 任务试验：

```javascript
var n_back_trial = {
  type: 'html-keyboard-response',
  stimulus: function() {
    if(sequence.length < how_many_back){
      var letter = jsPsych.randomization.sampleWithoutReplacement(n_back_set, 1)[0]
    } else {
      if(jsPsych.timelineVariable('match', true) == true){
        var letter = sequence[sequence.length - how_many_back];
      } else {
        var possible_letters = jsPsych.randomization.sampleWithoutReplacement(n_back_set, 2);
        if(possible_letters[0] != sequence[sequence.length - how_many_back]){
          var letter = possible_letters[0];
        } else {
          var letter = possible_letters[1];
        }
      }
    }
    sequence.push(letter);
    return '<span style="font-size: 96px;">'+letter+'</span>'
  },
  choices: ['M'],
  trial_duration: 1500,
  response_ends_trial: false,
  post_trial_gap: 500,
  data: {
    phase: 'test',
    match: jsPsych.timelineVariable('match')
  },
  on_finish: function(data){
    if(data.match == true){
      data.correct = (data.key_press != null)
    }
    if(data.match == false){
      data.correct = (data.key_press === null)
    }
  }
}

var n_back_trials = [
  {match: true},
  {match: false}
]

var n_back_sequence = {
  timeline: [n_back_trial],
  timeline_variables: n_back_trials,
  sample: {
    type: 'with-replacement',
    size: sequence_length,
    weights: [1, 2]
  }
}

timeline.push(n_back_sequence);
```

**说明：**
- `type: 'html-keyboard-response'`：指定这是一个键盘响应类型的试验。
- `stimulus`：定义显示给参与者的字母。
- `choices`：定义参与者可以选择的键（这里是 'M'）。
- `trial_duration`：定义试验的持续时间。
- `response_ends_trial`：指定响应是否结束试验。
- `post_trial_gap`：定义试验结束后的间隔时间。
- `data`：存储试验数据。
- `on_finish`：定义试验结束后的数据处理逻辑。

### 7. 添加实验反馈

在 `timeline` 数组中添加反馈步骤：

```javascript
var feedback = {
  type: 'html-keyboard-response',
  stimulus: function(){
    var test_trials = jsPsych.data.get().filter({phase: 'test'}).last(sequence_length-2);
    var n_match = test_trials.filter({match: true}).count();
    var n_nonmatch = test_trials.filter({match: false}).count();
    var n_correct = test_trials.filter({match: true, correct: true}).count();
    var false_alarms = test_trials.filter({match: false, correct: false}).count();

    var html = "<div style='width:800px;'>"+
      "<p>All done!</p>"+
      "<p>You correctly identified "+n_correct+" of the "+n_match+" matching items.</p>"+
      "<p>You incorrectly identified "+false_alarms+" of the "+n_nonmatch+" non-matching items as matches.</p>"
    return html;
  },
  choices: jsPsych.NO_KEYS
}
timeline.push(feedback);
```

**说明：**
- `type: 'html-keyboard-response'`：指定这是一个键盘响应类型的试验。
- `stimulus`：定义显示给参与者的反馈信息。
- `choices: jsPsych.NO_KEYS`：指定这个试验不需要键盘响应。

### 8. 初始化 JsPsych 实验

在脚本的最后，初始化 JsPsych 实验：

```javascript
jsPsych.init({
  timeline: timeline
})
```

**说明：**
- `jsPsych.init`：初始化 JsPsych 实验，传入 `timeline` 数组。

### 9. 运行实验

保存你的 HTML 文件，并在浏览器中打开它。你将看到一个 n-back 任务的实验，参与者需要按照指导语进行操作。

### 10. 显示结果

JsPsych 会在实验结束后显示结果。

希望这篇教程能帮助你顺利开始你的实验设计之旅！
