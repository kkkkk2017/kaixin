---
layout: post
title: Creating repetitive Task in Qualtrics using JS
---

Many user studies contain tasks that have similar structures but different contents. Most people implement this task by copying and pasting numerous blocks. 
But this method makes debugging very difficult, especially for someone like, me, who always makes minor mistakes.
This tutorial provides a guide to Qualtrics users to build such repetitive tasks without the tedious copying and pasting. 


We will use the "[Loop and Merge]:https://www.qualtrics.com/support/survey-platform/survey-module/block-options/loop-and-merge/" function in Qualtrics. 
### Step 1: create a multiple choice questions outside the loop block
### Step 2: select loop based on that question
### Step 3: add Fields by opening this Table
### Step 4: select display all fields, and randomize loop order
    To access the field in the Loop Table, you can use: 
        {lm://Field/1}
    To access the elements of loop, you can use: 
        ${lm://CurrentLoopNumber} or ${lm://TotalLoops}

### Done! Wait, you want to add conditions for the tasks?

## Assigning Condition to the tasks
Here I have two kinds of tasks randomly assigned to the participants. 
And I want two conditions evenly appear to each participant.
To do this, we are going to use JavaScript and Qualtrics Embedded Data: https://www.qualtrics.com/support/survey-platform/survey-module/survey-flow/standard-elements/embedded-data/ .

### Step 1: First, create two new Embedded Data and initiate them with values.
Here I have three Embedded Data, '_currentCond_' which stores the condition for the current loop,
'_totalCond1_' which stores the total appearance of condition 1, and '_totalCond2_' which stores the total appearance for condition 2.
### Step 2: select a question where to generate the conditions, select JavaScript icon, and put the following code:
```javascript
    Qualtrics.SurveyEngine.addOnload(function() {
    //get the total loop count and divide by two, which is the expected total appearance for each condition.
    var total_count = "${lm://TotalLoops}";
    total_count /= 2;

    var totalCond1 = parseInt(Qualtrics.SurveyEngine.getEmbeddedData('totalCond1'));
    var totalCond2 = parseInt(Qualtrics.SurveyEngine.getEmbeddedData('totalCond2'));

    var current_cond;

    if (totalCond1 < total_count && totalCond2 < total_count) {
        current_cond = Math.floor(Math.random() * 2); //randomly generate the contion
    } else if (totalCond1 == total_count) {
        current_cond = 1; //if all condition 1, then assign condition 2
    } else if (totalCond2 == total_count) {
        current_cond = 0; //if all condition 1, then assign condition 2
    } else {
        alert('count error');
    }
    //save the current condition as Embedded Data
    Qualtrics.SurveyEngine.setEmbeddedData('currentCond', current_cond);

    //update the countings
    if (current_cond == 0) {
        totalCond1 += 1;
        Qualtrics.SurveyEngine.setEmbeddedData('totalCond1', totalCond1);
    } else if (current_cond == 1) {
        totalCond2 += 1;
        Qualtrics.SurveyEngine.setEmbeddedData('totalCond2', totalCond2);
    } else {
        alert('condition error', current_cond);
    }
    });
```

### Step 3: Create a new question, and add the display logic. Select Embedded Data -> currentCond and value.
### Done!


## Creating Audio Tasks dynamically

### First, you need to host your audio files online. 
Please refer to this tutorial for hosting audio on Github: https://kywch.github.io/jsPsych-in-Qualtrics/github-pages/ .

### Step 2. In the Loop Table, add a field for the audio id.
Unfortunately, Qualtrics does not allow to change the field name. 

### Step 3. create a new question for the audio, and put the following html code in the question HTML view.
```html
    <audio id="player" controls> <source type="audio/mpeg">
    Your browser does not support the audio element. < /audio>
    <!--or remove the `controls' if you don't allow the participants to control audio-->
```
or if you want only need one audio,
```html
    <audio id="player" controls> <source type="audio/mpeg" src=[your_audio_url]>
    Your browser does not support the audio element. < /audio>
```

### Step 4. If you want the audio generated dynamically, put the following JS code. 
```javascript
    //Javascript
    Qualtrics.SurveyEngine.addOndisplay(function() {
    var task_github = [audio_url];
    var audio_name = '${lm://Field/5}';

    if (audio_name == '') {
        alert('audio name error');
    }
    var url = task_github + audio_name + '.mp3';

    var audio = document.getElementById("player");
    audio.src = url;
    audio.load();
    
    //if auto play
    audio.play();
    // I would wait for 1 second so the participant get ready for the audio.
    var timerInterval = setInterval(audio.play(), 1000);
    
    });
```

### Extra: keep on track of participants behaviours for the audio
```javascript
    audio.addEventListener('play', function() {
	
	});

	audio.addEventListener('pause', function() {	
		
	});
	
	audio.addEventListener("ended", function() {
    	
	});
```
or get the audio play time. 

```javascript
    audio.duration
    audio.currentTime
```

## Timestamp
I want to get the timestamp for each question, and download it at the end of the experiment.
```javascript
    //var timestamp = new Date().toLocaleString('en-AU', { timeZone: 'Australia/Sydney' });
	var timestamp = Date.now().toString(); //save the timestamp as a string
    var current = '${lm://CurrentLoopNumber}';
	  
	 if (!window.timestamps) {
		window.timestamps = [];
	 }
	 window.timestamps.push({ questionID: 'Q_'+current, timestamp: timestamp });
```
**'_window_' is a global variable.**

At the end of the survey:
```javascript
	var fileData = "Question,Timestamp\n";
	for (var i = 0; i < window.timestamps.length; i++) {
		var entry = window.timestamps[i];
		fileData += entry.questionID + "," + entry.timestamp + "\n";
	}
	
	// Create a download link
	var downloadLink = document.createElement('a');
	downloadLink.href = 'data:text/csv;charset=utf-8,' + encodeURIComponent(fileData);
	downloadLink.download = 'timestamps.csv';
	downloadLink.click(); //this is for auto download
	//if not auto,
	//downloadLink.innerHTML = 'Download Timestamps';
	//document.getElementById('QID119').appendChild(downloadLink);

```

## Something Extra: 
if you want to create the questions more dynamically using JS.

```html
<div id="Qtitle"></div>
<div id="answer"></div>
```
Generate Read or Listen Tasks

```javascript
Qualtrics.SurveyEngine.addOnload(function()
{
	/*Place your JavaScript here to run when the page loads*/
	var current_cond = Qualtrics.SurveyEngine.getEmbeddedData( 'currentCond'); 
	var header = document.getElementById("Qtitle");
	var answer = document.getElementById("answer");
	
	if (current_cond == 1){ 
		// listen
		header.textContent = "Please listen to this:";

		var audioElement = document.createElement("audio");
		audioElement.setAttribute("id", "player");
		audioElement.setAttribute("src", [your_audio_url]);
		audioElement.controls = true;

		answer.appendChild(audioElement);

		var audio = document.getElementById("player");
		audio.load();

	}
	else if (current_cond == 0) {
		header.textContent = "Please read this :";
		var content = "${lm://Field/3}";
		content = content.replace(/\r?\n|\r/g, "");
		answer.textContent = content;
	} else {
		alert("condition error");
	
	}
});

// remember to clean the content after submitting the question. 
Qualtrics.SurveyEngine.addOnUnload(function()
{
	/*Place your JavaScript here to run when the page is unloaded*/
	var header = document.getElementById("Qtitle");
	console.log(header);
	header.textContent = '';
	var answer = document.getElementById("answer");
	answer.textContent = '';
	console.log(answer);
	
});
```

Generate Speak or Type Tasks.
```html
<div id="Qtitle"></div>
<div id="answer"><button id="'mybutton">Record</button></div>
<!--This is a fake button-->
```

```javascript
Qualtrics.SurveyEngine.addOnload(function()
{
	/*Place your JavaScript here to run when the page loads*/
	var output_cond = parseInt(Qualtrics.SurveyEngine.getEmbeddedData( 'outputCondition')); 
	
	var header = document.getElementById("Qtitle");
	var answer = document.getElementById("answer");
	var mybutton = document.getElementById("mybutton");
	
	if (output_cond == 2){ 
		// write
		mybutton.style.display = "none";
		
		header.textContent = "Please type in the following textbox.";
		var textFieldElement = document.createElement("input");
  		textFieldElement.type = "text";
		textFieldElement.id = "textinput";
		answer.appendChild(textFieldElement);
		
        // save the textfield input in the Embedded Data
		Qualtrics.SurveyEngine.addOnPageSubmit(function() {
			Qualtrics.SurveyEngine.setEmbeddedData( 'answer', textFieldElement.value); 
		});
		
	}
	else if (output_cond == 3) {
		// speak
		mybutton.style.display = "inline-block";
		header.textContent = "Please click the record button and speak. ";
		// add event listener to track participants behaviuors
		mybutton.addEventListener("click", function() {
			if (mybutton.innerHTML == "Record"){
				mybutton.innerHTML  = "Stop";
			} else {
				mybutton.innerHTML  = "Record";
			}
  		});
		
	}
});
```

## If you want to a timer:
```javascript
var timer;

Qualtrics.SurveyEngine.addOnReady(function()
{
	/*Place your JavaScript here to run when the page is fully displayed*/
    this.hideNextButton(); //remember to hider the next Button
    
    timer = document.getElementById("mytimer");
    var secondsRemaining = 5; //5 seconds

    function updateTimer() {
        var minutes = Math.floor((secondsRemaining % 3600) / 60);
        var seconds = secondsRemaining % 60;

        // Format the time display as HH:MM:SS
        var timeString = formatDigits(minutes) + ":" + formatDigits(seconds);
        timer.textContent = timeString;

        // Decrease the remaining seconds by 1
        secondsRemaining--;

        // Stop the timer when it reaches 0
        if (secondsRemaining < 0) {
            clearInterval(timerInterval);
            timer.textContent = "00:00"; // Optional: Display a message when the timer ends
        }
    }

    function formatDigits(value) {
        // Add leading zero if the value is less than 10
        return value < 10 ? "0" + value : value;
    }

    // Start the timer
    var timerInterval = setInterval(updateTimer, 1000);
    var that = this;
    // Here, the nextButton will appear when the time is up, instead of go to the next page.
    var timer_action = setTimeout(function() { jQuery("#NextButton").show(); }, secondsRemaining*1000+50);
    Qualtrics.SurveyEngine.addOnPageSubmit(function() { clearTimeout(timer_action); })
	
});

Qualtrics.SurveyEngine.addOnUnload(function()
{
	/*Place your JavaScript here to run when the page is unloaded*/
	if (timer){
		clearInterval(timer);
	}
});
```
To hide the next button:
```javascript
    this.hideNextButton();
```

To submit page: 
```javascript
    this.clickNextButton();
```

## Now you are all setup. Have a fun experiment time!

![_config.yml]({{ site.baseurl }}/images/config.png)

