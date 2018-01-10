---
layout: post
title:      "Grasping Nested Forms in Rails"
date:       2018-01-10 20:59:09 +0000
permalink:  grasping_nested_forms_in_rails
---


I thought it was just me, but upon hearing from several others the headache that nested forms introduces when learning Rails, I decided to step back and dive a little deeper into how they work and how Rails handles them. 

First, what is a nested form? Let’s use a Hiking Trail domain to walk us through this.  We will have a trail model and a hiker model.  A trail has many hikers and a hiker has many trails. When beginning to build forms with Rails, we start off slow.  A form for a new hiker will simply ask for the details of just the hiker such as the hiker's name. It would make sense though, to at the same time, also allow the user to enter trails that they have hiked. This is where nested forms come in.  Within the form for the hiker we are “nesting” another form to get information on all of the trails.  

Lets take a look at what this would look like in regular HTML.  Using erb templating, the abbreviated version looks something similar to this:

```
<form method="POST" action="/hikers">
  <h1>New Hiker</h1>
  <input name="authenticity_token" type="hidden" value="token_value">

  <label for="name">Hiker Name: </label>
  <input type="text" id="name" name="hiker[name]"><br><br>

  <fieldset>
    <legend>Select or add a new trail</legend>

    <% Trail.all.each do |trail| %>
      <input type="checkbox" name="hiker[trail_ids][]" value="<%= trail.id %>"> <%= trail.name %></input>
    <% end %>
    <br><br>

    <label for="new_trail">New Trails: </label>
    <input type="text" name="hiker[trails][][name]">
    <input type="text" name="hiker[trails][][name]">


  </fieldset><br><br>

  <input type="submit" value="Create Hiker">
</form
```

This nested form will iterate through the collection of trails already saved to the database and list them as checkboxes.  It will also give the user a few text boxes to create new trails.  What we are concerned with here are the name=“hiker[trail_ids][]" and name=“hiker[trails][ ][name]" attributes.  Why do we write it like this? This is where ActiveRecord comes in.  Active Record gives us several new methods with the has_many relationship.  For the checkboxes, here are two of importance:

* collection_singular_ids         #=> hiker.trail_ids (gets the ids)
* collection_singular_ids=      #=> hiker.trail_ids = [1,4,6] (sets the ids)

Active Record gives us the method of, in this case, trail_ids=, allowing us to grab the ids of trails selected and
automatically put those into the hiker’s list of trails.  The empty array of hiker[trail_ids][ ] creates an array to fill
in all the ids that are to be passed in.  

As for the new trails, we need to take another step further to make this work. Right now, if we were to fill out
this form, we would get an error like this: 

**Trail(#70153448125880) expected, got {"name"=>"Days River Trail"} which is an instance of ActiveSupport::HashWithIndifferentAccess(#70153393832120)**

Our error is that our form is trying to pass in a hash rather than an actual instance of a trail.  ActiveRecord
can’t bail us out here so in this case, we have to define our own method in the Hiker class that will take the trails hash, iterate through each trail, either find it and push it into the hikers trail array or build and save a new trail. 

```
def trails=(trails)
    trails.each do |trail|
      if new_trail = Trail.find_by(name: trail[:name])
        self.trails << new_trail
      else
        self.trails.build(name: trail[:name])
      end
    end
  end
```

And thats its for building a nested form with HTML! 

Knowing Rails however, you can be sure that there are a few shortcuts.  To start, we will rebuild the form with *form_for* instead. 

```
<h1>New Hiker</h1>
<%= form_for @hiker do |f| %>

  <%= f.label :name, "Hiker Name" %>
  <%= f.text_field :name %><br><br>

  <fieldset>
    <legend>Select or add a new trail</legend>

    <%= f.collection_check_boxes :trail_ids, Trail.all, :id, :name %>
    <br><br>

    <%= f.fields_for :trails do |trail_builder| %>
      <%= trail_builder.label :name, "New Trail:" %>
      <%= trail_builder.text_field :name %><br><br>
    <% end %>
  </fieldset><br><br>

  <%= f.submit %>

<% end %>

```


The collection_check_boxes returns to us a collection of checkboxes found in Trail.all.  The :trail_ids sets our name attribute (giving us hiker[trail_ids][ ] like in the html version.  The :id sets the value to be passed through and the :name sets the label for the checkbox. 

For the new trail form, the fields_for iteration we see needs a few more things to get us running than just the code we see here. 

- First, we set fields_for as an enumerable so that several new trail fields can be set. Just like with the hiker though, we do need to instantiate a few new trail instances in our controller.  The code below will give us two “New Trail” text boxes on our form.  We could always add more or just do one.

```
def new
@hiker = Hiker.new
@hiker.trails.build
@hiker.trails.build
end
```



- Secondly, the default setting in rails is for nested attribute updating to be turned off.  We need to tell the Hiker class to allow nested attribute updating using the macro: accepts_nested_attributes_for :trails.  This macro defines a method on the model. We could also define a custom writer method here if we would like to do some additional conditional testing since the macro gives us only very basic functionality.  For our simple application here though, the macro alone will work just fine. 

There are several other steps to ensuring that our app is running correctly such as requiring strong params and routing, but these steps above should clarify our nested forms problem.  Nest away!

