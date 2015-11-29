---
layout: post
color: green
title:  "Email Google form data as PDF"
date:   2014-04-22 18:58:35
categories: 
tags: [google app scripts, email, form, spreadsheet]
---

Google Apps Script is a cloud based scripting language for light-weight application development in the Google Apps platform. It gets executed in the Google Cloud essentially providing easy ways to automate tasks across Google products and third party services.

In this tutorial, we will be sending a Google Spreadsheet as a pdf attachment to an email.

* First open your spreadsheet from your [Google Drive](https://drive.google.com/#my-drive).
* In the menubar, click on ```Tools``` and then goto ```Script Manager```.

![Form view](http://i1051.photobucket.com/albums/s432/brijeshb42/ghost-blog/1.jpg)

* In the ```Script Manager```, click on New. A new tab will open with a panel to select the type of project.
In this new tab, click on ```Blank Project```.

![Google app scripts](http://i1051.photobucket.com/albums/s432/brijeshb42/ghost-blog/2.jpg)

* Now a Script Editor will open having a code written like this:

{% highlight javascript %}
function myFunction() {
}
{% endhighlight %}

* Rename the previous function or add a new function named ```onOpen```.
* This ```onOpen``` function will now run every time you open your corresponding spreadsheet.
* Change the ```opOpen``` function to this:

{% highlight javascript %}
function onOpen() {
	var sheet = SpreadsheetApp.getActiveSpreadsheet();
	var entries = [{
	name : "Send Data to Email",
	functionName : "sendToEmail"
	}];
	sheet.addMenu("Functions", entries);
};
{% endhighlight %}

* This function adds a new menu named ```Functions``` to your spreadsheet page with 1 submenu ```Send Data``` to ```Email```. Now if you open your spreadsheet page again, these menu items will get added to your spreadsheet menubar.

![Function view](http://i1051.photobucket.com/albums/s432/brijeshb42/ghost-blog/3.jpg)

* When you click on ```Send Data to Email```, the function ```sendToEmail``` will get executed.
* Now you have to define the ```sendToEmail``` function. In the script editor, create another function ```sendToEmail``` that looks like this:

{% highlight javascript linenos %}
function sendToEmail(){
	var name = Browser.inputBox('Enter email', Browser.Buttons.OK_CANCEL);
		if(name!=="cancel" && name!==""){
		var id = SpreadsheetApp.getActiveSpreadsheet().getId();
		var spreadsheetFile = DocsList.getFileById(id); 
		var blob = spreadsheetFile.getAs('application/pdf');
		MailApp.sendEmail(name, 'Subject', 'Some body text', {attachments:[blob]});
	}
}
{% endhighlight %}

* After adding this function, save your script using the menubar.

* In line 1, the variable ```name``` asks for an email from the user to which the email is to be sent.
* In line 2, ```name``` is checked to be not empty and contain a value.
* In line 3, variable ```id``` is assigned the unique id of your spreadsheet.
* In line 4, the spreadsheet file is retrieved from your google drive into ```spreadSheetFile```.
* In line 5, the spreadsheet is converted to pdf format for sending.
* In line 6, the file is now sent as an attachment to email id saved in ```name```. You can also add a subject or body to the email.
* You email id which you used to sign in to google drive will be used as your sender email.

Now when you refresh your spreadshhet page, an additional menu ```Function``` will be added with a submenu, ```Send Data to Email```.

![Dialog](http://i1051.photobucket.com/albums/s432/brijeshb42/ghost-blog/4.jpg)

When you click on this submenu, you will be prompted to enter an email. If you enter a valid email, the corresponding spreadsheet data will be sent as attachment to the entered email.
