# Rapidly Prototyping Web Applications With Meteor


### Installation
```
curl https://install.meteor.com | /bin/sh
```

### Creating a new app
```
meteor create contacts
cd contacts
meteor
```

### Packages
```
meteor list --using
meteor list
```

```
meteor add bootstrap
```


### CSS
```css
body {
	padding-top: 20px;
}

.contact_list > li {
	width: 230px;
	min-height: 250px;
	float: left;
	margin-right: 20px;
	margin-bottom: 20px;
	border-right: 1px solid #eee;
	border-bottom: 1px solid #eee;
}

.access_denied {
	margin: 100px auto;
}
```

### Header Template
```html
<body>
	<div class="container">
		<div class="row-fluid">
			{{> header}}
		</div>
	</div>
</body>
```


```html
<template name="header">
	<div class="span12">
		<div class="well well-small">
			<h1>
				Contacts App
			</h1>
		</div>
	</div>
</template>
```


### Creating a collection

```js
Contacts = new Meteor.Collection("contacts");
```


### Interacting with collections
```js
Contacts.insert({
	name: "bharani",
	email: "bharani@gmail.com",
 	phone: "9891836696",
});

Contacts.update({
	_id: "3chG6g9AaKB6zuDXu" 
}, { $set: 
{
	name: "Bharani Muthukumaraswamy"
}});


Contacts.find();
Contacts.find({name: "Bharani Muthukumaraswamy"});

Contacts.remove({ _id: "id string.." });
```

### New Contact Form
```html
<div class="row-fluid">
	<div class="span4">
		{{> add_contact}}
	</div>
</div>
```

```html
<template name="add_contact">
  <div class="well">
    <form>
      <label>Name</label>
      <input type="text" class="input-block-level name">

      <label>Email</label>
      <input type="text" class="input-block-level email">

      <label>Phone</label>
      <input type="text" class="input-block-level phone">

      <label>Twitter</label>
      <input type="text" class="input-block-level twitter">
      
      <hr>

      <input type="submit" class="btn btn-primary btn-large" value="Add Contact">
    </form>
  </div>
</template>
```


### Contact list
```html
<template name="contact_list">
	<div class="span8">
		<ul class="unstyled contact_list">
			{{#each contacts}}
				{{> contact_item }}
			{{/each}}
		</ul>
	</div>
</template>
```

```
<template name="contact_item">
    <li>
      <h3>{{name}}</h3>
      <ul class="unstyled">
        <li><strong>Email: </strong>{{ email }}</li>
        <li><strong>Phone: </strong>{{ phone }}</li>
        <li><strong>Twitter: </strong>{{ twitter }}</li>
      </ul>
    </li>
</template>
```





```js
if( Meteor.isClient ) {
  Template.contact_list.contacts = function() {
		return Contacts.find();
	}
}
```

### Adding new contacts
```
  Template.add_contact.events({
    "submit form": function(e, t) {
      e.preventDefault();

      var name = t.find(".name"),
          email = t.find(".email"),
          phone = t.find(".phone"),
          twitter = t.find(".twitter"),
          form = e.target;

      if(!name.value.length || !email.value.length) {
        return alert("Enter a name and a valid email id.");
      }

      Contacts.insert({
        name: name.value,
        email: email.value,
        phone: phone.value,
        twitter: twitter.value
      });

      form.reset();
    }
  });
```

### Deleting Contacts
```
<button class="btn btn-small edit">Edit</button>
<button class="btn btn-small btn-danger delete">Delete</button>
```

```
Template.contact_item.events({
	"click .delete": function(e, t) {
    	if(confirm("Are you sure?")){
        	Contacts.remove(t.data._id);
      	}
    }
});
```
### Editing Contacts

```
<template name="contact_item">
    <li>
      {{#if isEditing }}
        <input type="text" class="name" value="{{ name }}">
        <input type="text" class="email" value="{{ email }}">
        <input type="text" class="phone" value="{{ phone }}">
        <input type="text" class="twitter" value="{{ twitter }}">

        <button class="btn btn-small btn-primary update">Update</button>
        <button class="btn btn-small btn-inverse cancel">Cancel</button>


      {{ else }}
        <h3>{{name}}</h3>
        <ul class="unstyled">
          <li><strong>Email: </strong>{{ email }}</li>
          <li><strong>Phone: </strong>{{ phone }}</li>
          <li><strong>Twitter: </strong>{{ twitter }}</li>
        </ul>

        <button class="btn btn-small edit">Edit</button>
      {{/if}}
  
        <button class="btn btn-small btn-danger delete">Delete</button>
      
    </li>
</template>
```

```
 Template.contact_item.isEditing = function() {
    return Session.equals("currentlyEditing", this._id);
  }
```



```
"click .edit": function(e, t) {
      Session.set("currentlyEditing", t.data._id);
    },

    "click .update": function(e, t) {
      var name = t.find(".name"),
          email = t.find(".email"),
          phone = t.find(".phone"),
          twitter = t.find(".twitter");

      if(!name.value.length || !email.value.length) {
        return alert("Enter a name and a valid email id.");
      }

      Contacts.update({_id: t.data._id },{ $set: {
        name: name.value,
        email: email.value,
        phone: phone.value,
        twitter: twitter.value
      }});

      Session.set("currentlyEditing", false);
    },

    "click .cancel": function(e, t) {
      Session.set("currentlyEditing", false);
    }

```


### Pub/Sub
```
Meteor.autosubscribe(function() {
	Meteor.subscribe("contacts", Meteor.userId());
})
```

```
Meteor.publish("contacts", function() {
	return Contacts.find({ userId: this.userId });
})
```


### Allow/Deny

```
Contacts.allow({
	insert: function(userId, doc) {
		return !!userId
	},

	update: function(userId, doc) {
		if(userId && doc.userId == userId) return true;
		return false;
	},
	
	remove: function(userId, doc) {
		if(userId && doc.userId == userId) return true;
		return false;
	}
})
```

### App View (main.html) (after cleanup)
```
<head>
	<title>Contacts App</title>
</head>

<body>
	<div class="container">
		{{> header }}
		
		<hr>
		
		<div class="row-fluid">
			{{> app_view }}
		</div>
	</div>  
</body>

```


```
<template name="app_view">
	{{#if currentUser }}
		{{> add_contact}}			
		{{> contact_list }}
	{{else}}
		{{> access_denied }}
	{{/if}}
	
</template>
```


```
<template name="access_denied">
	<div class="row-fluid">
		<div class="span12">
			
		<h2 class="text-center access_denied">
				Welcome to our contacts application! <br>
				<small>
					Login or signup to proceed.
				</small>
			</h2>
			
		</div>
	</div>
</template>
```