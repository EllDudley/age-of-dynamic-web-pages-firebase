> TL; DR  Grab the final package from  https://github.com/webix-hub/webix-firebase

### Dynamic Web

Not so long ago web apps were slow and phlegmatic. The whole page had to be reloaded to update information. AJAX has only slightly improved this situation. Now we update separate blocks instead of reloading the whole page. Hardly had we got used to this technique, when a new web transformation appeared on the scene. It is dynamical web, where the content of pages updates by itself.

Now you can find such elements on Facebook or Twitter pages, for example. Data are changed there without our participation. As soon as someone has added a comment or re-tweeted the post, the changes are reflected on the page immediately.

There are several technologies behind dynamical web. WebSockets are the most important of them. (You can read how to use them with Webix [here](http://webix.com/blog/creating-a-web-chat-with-webix-and-websocket-api/)). However, web sockets present a very low level of abstraction. Fortunately, it's possible to get all the benefits of dynamic updates without diving so deeply into the details of data synchronization. There are several great solutions created by web enthusiasts. They allow avoiding difficulties and creating cool dynamic apps.

### FireBase


[FireBase ](https://www.firebase.com/) is the online database that can be used to build real time mobile and web apps within minutes by using client-side code only. 

Let's look how it works in practice:

```js
<script>
      var collection= new Firebase('https://webix-demo.firebaseio.com/books');
      collection.on('child_added', function(snapshot) {
        [MESSAGE CALLBACK CODE GOES HERE]
      });
</script>
```

This code defines some data collections and registers the handler that will be called each time new data are added. That's it. In the world of dynamic web you don't read data, but set the rules that define what to do when data are added or changed. 

You can check live demo and read more [in the official FireBase Tutorial](https://www.firebase.com/tutorial/#tutorial/basic/0)

Also, to get your own online DB you need to register at http://firebase.com. They have a free plan which is more than enough for making simple apps and prototyping. 


### FireBase + Webix

FireBase provides a nice API, so integration with Webix is quite simple. The straight approach is to use the above described **child_added** event and call the`add` method of the component. 

```js
myDataRef.on('child_added', function(data){
    var obj = data.val();
	obj.id = data.key();

    //place Webix API
	someView.add(obj);
});
```

The important thing is that we have assigned **data.key()** to the **obj.id**. Webix and FireBase use different approaches for ID storing. FireBase has a special **key** method to retrieve the id, while Webix expects to find the "id" property directly in the data object. 

The above snippet is fully working but it has one problem - it is slow. The component will repaint itself each time new data are added. To fix it, we can use a different Firebase event called "value". This event occurs when data are updated in the collection and provides all data of collection at once. 

```js
collection.once("value", function(data){
	var source = data.val();
	var result = [];
	for (var key in source){
		var record = source[key];
		record.id = key;
		result.push(record);
	}
	
	table.parse(data);
});
```

In the above example, instead of calling `add` for each record we are waiting until all records from this collection are received and then use the `parse` method for loading data. 

In a real app you will have to handle both `value` and `child_added` events, as a component needs to load the initial data and add new records that will be added to the data collection after initial rendering. Actually, you will need to handle two more events as well: the`child_changed`event and the`child_removed`one. These events will occur after data changing and data deleting, respectively. 


Let's look at data saving now: 
```
someView.attachEvent("onAfterAdd", function(id){
    collection.push(obj.data);
});
```

Once again, the code is rather simple. When some data have been added to the component, we are calling the related methods of FireBase API. Similar to adding operation we can write the code for deleting and editing. 

Thus, we have a fully working two-way data syncing between FireBase and Webix component. Changes in Component are saved back to FireBase and are synced among all clients. Nice. Still, quite a lot of code needs to be written each time.  We can do better. Instead of writing a bunch of code each time we can wrap the code in proxy objects and write only minimum necessary syntax. 

```js
webix.ui({
	view:"datatable",
	autoConfig:true,
	url: "firebase->books",
	save:"firebase->books"
})
```

The above code looks much better, doesn't it? All complexity of data syncing is hidden in the firebase proxy object which looks like this:

```js
webix.firebase = new Firebase("https://webix-demo.firebaseio.com/");

webix.proxy.firebase = {
  $proxy:true,
  load:function(view, callback){
    //decode string reference if necessary
    if (typeof this.source == "object")
      this.collection = this.source;
    else
      this.collection = this.collection || webix.firebase.child(this.source);

    // ------------------------------ //
    ... all data loading code here ...
    // ------------------------------ //
  },
  save:function(view, obj, dp, callback){
    //decode string reference if necessary
    if (typeof this.source == "object")
      this.collection = this.source;
    else
      this.collection = this.collection || webix.firebase.child(this.source);

  // ------------------------------ //
  ... all data saving code here ...
  // ------------------------------ //
  }
};
```

If you are interested, here is [the full code of the proxy object](https://github.com/webix-hub/webix-firebase/blob/master/codebase/webix-firebase.js)

As you can see, we have wrapped our code in a custom object. All data loading logic goes to the **load** method, all data saving logic goes to the **save** method. 

In both methods we decode the parameter of proxy object. If it is a string (name of collection) we convert it to collection object. 

Also, for the above code to work, we need to assign the firebase instance to the **webix.firebase** property. It is necessary only if you use string urls.

### Results

You can download the latest version of the library from github https://github.com/webix-hub/webix-firebase or install it through bower  `bower install webix-firebase`

The library can be used to make a one-way or two way data-binding between FireBase and Webix components (datatable, list, dataview, chart, etc.). 

You can also check [demos](https://github.com/webix-hub/webix-firebase#samples) and [technical docs](https://github.com/webix-hub/webix-firebase). 

The library is available under the MIT license, so you can use it freely.
Have fun!

**Author: Maksim Kozhukh**