---
layout: post
title:      "Building a Rails App with jQuery and JS Frontend"
date:       2018-02-25 19:44:05 +0000
permalink:  building_a_rails_app_with_jquery_and_js_frontend
---


Implementing jquery and javascript functionality into the front end of a Rails Application really starts to make myself as a developer feel like I can create sites for the 21st century.  

From the beginning of design of this site, I knew that I wanted to take more time to build something that I could see someday being developed further into a site that hikers might actually want to use.  The app allows users to interact with Michigan hiking trails by either creating lists of hiking trails, marking trails as having been hiked, asking questions about trails, answers questions and writing tips about trails.  They can also submit their own trails.  

### The Trails Show page with jquery and javascript:

The trails show page implements a lot of the functionality of the site as a whole.  Here a user can views details about a specific trail.  They can also mark the trail as having been hiked before by the user.  The button click triggers a basic replacement of HTML that removes the button and shows text of "Hiked".  The second half of the page allows users to either post a tip about the trail, ask a question or respond to a question with an answer.  These three function utilize similar processes of submitting small forms through POST requests to the appropriate route and straight away appending the data to the DOM.  The requests are serialized through specific Serializers that format that data in json.


![Screetshot of the a Trail Show Page](https://i.imgur.com/k2o2tYj.png)

### The Questions and Answers with jquery and javascript:

I wanted to utilize a next button so that users could sift through questions that had been posted about any trail with the corresponding answers.  The next button sends a GET request to the next route in the questions controller which then calls the next question through a model method.  The next question is rendered as json with help from the QuestionSerializer which the javascript function is then able to utilize to format the question and answers through constructors and prototype functions.  

One interesting bit I learned when using the serializers was that I could crate a method to return specific data.  The issue I had was that Serializers by nature do not allow for deeply nested associations but I needed to return the user of the answer of the question.  The Question couldn't access the user of the answer, just the answer and it's user_id.  I was able to create a custom method like this to allows for the user data to be returned: 

```
def answers
    custom_answers = []
    object.answers.each do |answer|
      custom_answer = answer.attributes

      custom_answer[:username] = answer.user.username
      custom_answer[:created_at] = answer.created_at
      custom_answer[:title] = answer.title

      custom_answers.push(custom_answer)
    end

    return custom_answers
  end
	```
	
	I then included :answers as an attribute which returned JSON like this: 
	
```	"answers":[ {"id":17,"title":"Yes the views are spectacular.","question_id":16,"user_id":3,"created_at":"2018-02-25T18:04:33.443Z","updated_at":"2018-02-25T18:04:33.443Z","unread":true,"username":"Amanda Ackerman"}], ```
	
, along with the rest of the question and trail information.  


The github repository for this project can be viewed here: [Trailblazer](https://github.com/ackerm44/trailblazer)
