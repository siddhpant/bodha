---
title: Has anyone solved the makkhichoose puzzle?
date: 2016-04-05
tags: ["challenge", "programming", "js", "migrated", "quora"]
---

Well, they use that puzzle for hiring, so I won't provide the answer here, or their purpose would be defeated.

But nevertheless, I will provide you a hint. Look at the JS file in POST data.

Confused? Can't find? Here it is!

{{< code language="js" expand="Show" collapse="Hide" isCollapsed="true" >}}
function sendData(in_data,dest_url) {
    var req_send = $.ajax({
            url: dest_url,
            type: 'POST',
            contentType: 'application/json',
            data: in_data,
            dataType: 'json'
        });
	return req_send;
		
}; //sendData



function munchClues(data) {
	
	var makkhi_link= '<a href="MakkhiChoose" target="_blank" title="MakkhiChoose browser plug-in, open in new window">MakkhiChoose</a>';
	var econ_link = '<a href="MakkhiChoose - Tricks and extensions to enhance your browsing experience - The Economic Times" target="_blank" title="Press Coverage in The Times of India">1.5 Lakh Indians</a>.';
	var disp_arr=[ "Hey you! There's a puzzle in here. Click on the arrow for more clues.", 
			"This is a coding puzzle. You'll need to write some code.",
			"It isn't tough. The clues are all here, in this page",
			"But you'll have to look closely.",
			"It might not even be in these sentences.",
			"Look under the covers, as any good detective would",
			"Once you get the secret phrase, type it in the box to the left",
			"Who are we? We are the makers of "+makkhi_link,
			"Dont have it? Install "+makkhi_link+" and you'll get one more BIG clue!",
			"The best way to save while shopping online, according to more than "+econ_link,
			"<b>You</b> can help us make it better, if you are good enough to crack this puzzle",
			"OK, that's all the help you'll get. Good luck!"]
	
	var disp_msg;
	
	if (data['i']){
		var next_val = parseInt(data['n']);
		$('#riddle').attr('index',next_val-1);	
		$('#riddle').attr('next',next_val);	
		$('#riddle').attr('word',data['w']);
		$('#riddle').html(disp_arr[(next_val-1)%12]);
	
		if (next_val%5==0) {
			$('#riddle').attr('message','Here is another hint for you: it involves some prime numbers. But not in a straightforward way.');
		} else {
			$('#riddle').removeAttr('message');
	
		}
	
	}
	else {
		$('#riddle').attr('index',0);	
		$('#riddle').attr('next',1);	
		$('#riddle').attr('word','right');
		$('#riddle').attr('message','try numbers between 0 and 197');
	
		disp_msg= "Hey you! There's a puzzle in here. Click on the arrow for more clues.";
		$('#riddle').html(disp_msg);


	}

}

function clueFail(data) {
	disp_msg= "Whoops! Lots of puzzlers here. Can't reach the server, sorry. Try later";
	$('#riddle').html(disp_msg);
}


function phraseFail(data) {
	disp_msg= "Whoops! Lots of puzzlers here. Can't load the next clue, sorry. Try later";
	typeAns(disp_msg);
}


function submitN() {
	switchMCImage();
	var i_val = $('#riddle').attr('next');
	req_n = sendData(JSON.stringify({'i':i_val}),'/clue');
	req_n.done(munchClues);
	req_n.fail(clueFail);
}




function munchPhrase(data) {

	if (data['i']){
		typeAns(data['m']);
		$('#riddle').remove();
		$('#nBut').remove();
		$('#askr').remove();
	}
	else {
		var wrogn_kholi = Math.floor((Math.random()*16)+1);
		if (wrogn_kholi<8) {
		typeAns("Sorry, that isn't it.");
		}
		else {
			typeAns(data['m']);
		}
	}

}//munchPhrase


function submitQ() {

	if ($('#answr').hasClass('typing')) {
		return true;
	}


	$('#answr').addClass('typing');
	
	var sendQuest = true;
	var q_val = $.trim($('textarea').val()).toLowerCase();
	var answer_val;



	if ((q_val == '') ||  (q_val=='type in the secret phrase...')){
		answer_val = "That isn't it. Have you tried to solve the puzzle?";
		sendQuest = false;
		typeAns(answer_val);
	}
	
	else if ((q_val.match(/shit/gi)) || (q_val.match(/fuck/gi)) || (q_val.match(/dumb/gi)) || (q_val.match(/whore/gi))) {
		answer_val = "Hey now! No reason to get snappy. Wash your mouth with soap water";
		sendQuest = false;
		typeAns(answer_val);
	
	}
	
	else if ((q_val.match(/cricket|baby|ipad|varun|windows|nokia|lumia/gi))) {
		answer_val = "No, the clues are not in the images. This is a coding puzzle.";
		sendQuest = false;
		typeAns(answer_val);
	
	}
	
	else if ((q_val.match(/aaa/gi)) || (q_val.match(/jjj/gi)) || (q_val.match(/ooo/gi)) || (q_val.match(/abc/gi)) || (q_val.match(/sfg/gi)) || (q_val.match(/jkjk/gi)) || (q_val.match(/sdj/gi)) || (q_val.match(/sdg/gi)) || (q_val.match(/jhg/gi)) || (q_val.match(/asd/gi)) || (q_val.match(/\d/gi)) || (q_val.match(/puz/gi)) || (q_val.match(/rid/gi)) || (q_val.match(/que/gi)) || (q_val.match(/fuc/gi)) || (q_val.match(/shi/gi))) {
		answer_val = "You are typing in random strings. That's a good strategy -- if you have a team of a hundred thousand monkeys";
		sendQuest = false;
		typeAns(answer_val);
		}
	
	else {


​		
		req_phrase = sendData(JSON.stringify({'q':q_val}),'/phrase');
		req_phrase.done(munchPhrase);
		req_phrase.fail(phraseFail);
	
	}




} //submitQ



function typeAns(answer_val) {

	$('#answr').empty();
	
	var wordArray = answer_val.split(' '), i = 0;
	
	INV = setInterval(function () {
		if (i >= wordArray.length - 1) {
			clearInterval(INV);
			$('textarea').val('');
			$('#answr').removeClass('typing');
			switchMCImage();
		}
	$('#answr').append(wordArray[i] + ' ');
	    i++;
	}, 200);

} //typeAns


function switchMCImage() {

	var show_logo_int = Math.floor((Math.random()*6)+1);
	
	$('#makkhiimg').attr('src','Makkhi/files/makkhi_alt_'+show_logo_int+'.png')

} //switchMCImage



function typeAnswer(answer_val) {

	var wrdArray = answer_val.split(' '), i = 0;
	return function(){
		if (i >= wrdArray.length - 1) {
	            clearInterval(INV);
	    	}
	    	$el.append(wrdArray[i] + ' ');


	} //closure


} //typeAnswer

var orig_img_src;

$(document).ready(function(){

orig_img_src = $('#rgimg').attr('src');

$('textarea').one('click',function(){$(this).html('');});

$('#qBut').click(submitQ);


$('#nBut').click(submitN);


$('textarea').keypress(function(e){
  if(e.keyCode == 13 && !e.shiftKey) {
   e.preventDefault();
	if ($('#answr').hasClass('typing')) {
		//do nothing
	}
	else {
	submitQ();
	}
  }
});

});//documentready
{{< /code >}}

Now I won't say anything more. Rest is up to you.

If you are scratching your head for the other puzzle (which is fun & a LOT easier), then I will provide a solution for it here.

You need to put off the javascript timer of the website.

To do that, just enter this in the URL bar :

```js
javascript:window.setTimeout = function(func, delay) {};window.setInterval = function(func, delay) {};document.getElementById("timer").innerHTML="200";
```

Now you have 200 seconds for that. And if you have difficulty finding the red box, then hold Ctrl+A and solve the puzzle.

Thanks for reading!
