# How to Create a PHP/MySQL Powered Forum From Scratch

[How to Create a PHP/MySQL Powered Forum From Scratch](https://code.tutsplus.com/tutorials/how-to-create-a-phpmysql-powered-forum-from-scratch--net-10188)

by Evert Padje17 Mar 2010

In this tutorial, we're going to build a PHP/MySQL powered forum from scratch. This tutorial is perfect for getting used to basic PHP and database usage. If you need extra help with this or any other PHP issues, try contacting one of the PHP developers on Envato Studio. They can help you with everything from PHP fixes to developing robust PHP applications.

## Step 1: Creating Database Tables

It's always a good idea to start with creating a good data model when building an application. Let's describe our application in one sentence: We are going to make a forum which has users who create topics in various categories. Other users can post replies. As you can see, I highlighted a couple of nouns which represent our table names.

### 1.1 Users

Categories

Topics

Posts

These three objects are related to each other, so we'll process that in our table design. Take a look at the scheme below.

图图

Looks pretty neat, huh? Every square is a database table. All the columns are listed in it and the lines between them represent the relationships. I'll explain them further, so it's okay if it doesn't make a lot of sense to you right now.

I'll discuss each table by explaining the SQL, which I created using the scheme above. For your own scripts you can create a similar scheme and SQL too. Some editors like MySQL Workbench (the one I used) can generate .sql files too, but I would recommend learning SQL because it's more fun to do it yourself. A SQL introduction can be found at W3Schools.

### 1.2 Users Table

```
CREATE TABLE users (
user_id     INT(8) NOT NULL AUTO_INCREMENT,
user_name   VARCHAR(30) NOT NULL,
user_pass   VARCHAR(255) NOT NULL,
user_email  VARCHAR(255) NOT NULL,
user_date   DATETIME NOT NULL,
user_level  INT(8) NOT NULL,
UNIQUE INDEX user_name_unique (user_name),
PRIMARY KEY (user_id)
) TYPE=INNODB;
```

The CREATE TABLE statement is used to indicate we want to create a new table, of course. The statement is followed by the name of the table and all the columns are listed between the brackets. The names of all the fields are self-explanatory, so we'll only discuss the data types below.

user_id
"A primary key is used to uniquely identify each row in a table."
The type of this field is INT, which means this field holds an integer. The field cannot be empty (NOT NULL) and increments which each record inserted. At the bottom of the table you can see the user_id field is declared as a primary key. A primary key is used to uniquely identify each row in a table. No two distinct rows in a table can have the same value (or combination of values) in all columns. That might be a bit unclear, so here's a little example.

There is a user called John Doe. If another users registers with the same name, there's a problem, because: which user is which? You can't tell and the database can't tell either. By using a primary key this problem is solved, because both topics are unique.

All the other tables have got primary keys too and they work the same way.

user_name
This is a text field, called a VARCHAR field in MySQL. The number between brackets is the maximum length. A user can choose a username up to 30 characters long. This field cannot be NULL. At the bottom of the table you can see this field is declared UNIQUE, which means the same username cannot be registered twice. The UNIQUE INDEX part tells the database we want to add a unique key. Then we define the name of the unique key, user_name_unique in this case. Between brackets is the field the unique key applies to, which is user_name.

user_pass
This field is equal to the user_name field, except the maximum length. Since the user password, no matter what length, is hashed with sha1(), the password will always be 40 characters long.

user_email
This field is equal to the user_pass field.

user_date
This is a field in which we'll store the date the user registered. It's type is DATETIME and the field cannot be NULL.

user_level
This field contains the level of the user, for example: '0' for a regular user and '1' for an admin. More about this later.

Categories Table

```
CREATE TABLE categories (
cat_id          INT(8) NOT NULL AUTO_INCREMENT,
cat_name        VARCHAR(255) NOT NULL,
cat_description     VARCHAR(255) NOT NULL,
UNIQUE INDEX cat_name_unique (cat_name),
PRIMARY KEY (cat_id)
) TYPE=INNODB;
```

These data types basically work the same way as the ones in the users table. This table also has a primary key and the name of the category must be an unique one.

Topics Table

```
CREATE TABLE topics (
topic_id        INT(8) NOT NULL AUTO_INCREMENT,
topic_subject       VARCHAR(255) NOT NULL,
topic_date      DATETIME NOT NULL,
topic_cat       INT(8) NOT NULL,
topic_by        INT(8) NOT NULL,
PRIMARY KEY (topic_id)
) TYPE=INNODB;
```

This table is almost the same as the other tables, except for the topic_by field. That field refers to the user who created the topic. The topic_cat refers to the category the topic belongs to. We cannot force these relationships by just declaring the field. We have to let the database know this field must contain an existing user_id from the users table, or a valid cat_id from the categories table. We'll add some relationships after I've discussed the posts table.

Posts Table

```
CREATE TABLE posts (
post_id         INT(8) NOT NULL AUTO_INCREMENT,
post_content        TEXT NOT NULL,
post_date       DATETIME NOT NULL,
post_topic      INT(8) NOT NULL,
post_by     INT(8) NOT NULL,
PRIMARY KEY (post_id)
) TYPE=INNODB;
```

This is the same as the rest of the tables; there's also a field which refers to a user_id here: the post_by field. The post_topic field refers to the topic the post belongs to.

"A foreign key is a referential constraint between two tables. The foreign key identifies a column or a set of columns in one (referencing) table that refers to a column or set of columns in another (referenced) table."
Now that we've executed these queries, we have a pretty decent data model, but the relations are still missing. Let's start with the definition of a relationship. We're going to use something called a foreign key. A foreign key is a referential constraint between two tables. The foreign key identifies a column or a set of columns in one (referencing) table that refers to a column or set of columns in another (referenced) table. Some conditions:

The column in the referencing table the foreign key refers to must be a primary key
The values that are referred to must exist in the referenced table
By adding foreign keys the information is linked together which is very important for database normalization. Now you know what a foreign key is and why we're using them. It's time to add them to the tables we've already made by using the ALTER statement, which can be used to change an already existing table.

We'll link the topics to the categories first:

1
ALTER TABLE topics ADD FOREIGN KEY(topic_cat) REFERENCES categories(cat_id) ON DELETE CASCADE ON UPDATE CASCADE;
The last part of the query already says what happens. When a category gets deleted from the database, all the topics will be deleted too. If the cat_id of a category changes, every topic will be updated too. That's what the ON UPDATE CASCADE part is for. Of course, you can reverse this to protect your data, so that you can't delete a category as long as it still has topics linked to it. If you would want to do that, you could replace the 'ON DELETE CASCADE' part with 'ON DELETE RESTRICT'. There is also SET NULL and NO ACTION, which speak for themselves.

Every topic is linked to a category now. Let's link the topics to the user who creates one.

1
ALTER TABLE topics ADD FOREIGN KEY(topic_by) REFERENCES users(user_id) ON DELETE RESTRICT ON UPDATE CASCADE;
This foreign key is the same as the previous one, but there is one difference: the user can't be deleted as long as there are still topics with the user id of the user. We don't use CASCADE here because there might be valuable information in our topics. We wouldn't want that information to get deleted if someone decides to delete their account. To still give users the opportunity to delete their account, you could build some feature that anonymizes all their topics and then delete their account. Unfortunately, that is beyond the scope of this tutorial.

Link the posts to the topics:

1
ALTER TABLE posts ADD FOREIGN KEY(post_topic) REFERENCES topics(topic_id) ON DELETE CASCADE ON UPDATE CASCADE;
And finally, link each post to the user who made it:

1
ALTER TABLE posts ADD FOREIGN KEY(post_by) REFERENCES users(user_id) ON DELETE RESTRICT ON UPDATE CASCADE;
That's the database part! It was quite a lot of work, but the result, a great data model, is definitely worth it.

## Step 2: Introduction to the Header/Footer System

Each page of our forum needs a few basic things, like a doctype and some markup. That's why we'll include a header.php file at the top of each page, and a footer.php at the bottom. The header.php contains a doctype, a link to the stylesheet and some important information about the forum, such as the title tag and metatags.

header.php

```php
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN"
"http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">
<html xmlns="http://www.w3.org/1999/xhtml" xml:lang="nl" lang="nl">
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
    <meta name="description" content="A short description." />
    <meta name="keywords" content="put, keywords, here" />
    <title>PHP-MySQL forum</title>
    <link rel="stylesheet" href="style.css" type="text/css">
</head>
<body>
<h1>My forum</h1>
    <div id="wrapper">
    <div id="menu">
        <a class="item" href="/forum/index.php">Home</a> -
        <a class="item" href="/forum/create_topic.php">Create a topic</a> -
        <a class="item" href="/forum/create_cat.php">Create a category</a>
         
        <div id="userbar">
        <div id="userbar">Hello Example. Not you? Log out.</div>
    </div>
        <div id="content">
```

The wrapper div will be used to make it easier to style the entire page. The menu div obviously contains a menu with links to pages we still have to create, but it helps to see where we're going a little bit. The userbar div is going to be used for a small top bar which contains some information like the username and a link to the logout page. The content page holds the actual content of the page, obviously.

The attentive reader might have already noticed we're missing some things. There is no </body> or </html> tag. They're in the footer.php page, as you can see below.

```html
</div><!-- content -->
</div><!-- wrapper -->
<div id="footer">Created for Nettuts+</div>
</body>
</html>
```

When we include a header and a footer on each page the rest of the page get embedded between the header and the footer. This method has got some advantages. First and foremost, everything will be styled correctly. A short example:

```php
<?php
$error = false;
if($error = false)
{
    //the beautifully styled content, everything looks good
    echo '<div id="content">some text</div>';
}
else
{
    //bad looking, unstyled error :-( 
} 
?>
```

As you can see, a page without errors will result in a nice page with the content. But if there's an error, everything looks really ugly; so that's why it's better to make sure not only real content is styled correctly, but also the errors we might get.

Another advantage is the possibility of making quick changes. You can see for yourself by editing the text in footer.php when you've finished this tutorial; you'll notice that the footer changes on every page immediately. Finally, we add a stylesheet which provides us with some basic markup - nothing too fancy.

```css
body {
    background-color: #4E4E4E;
    text-align: center;         /* make sure IE centers the page too */
}
 
#wrapper {
    width: 900px;
    margin: 0 auto;             /* center the page */
}
 
#content {
    background-color: #fff;
    border: 1px solid #000;
    float: left;
    font-family: Arial;
    padding: 20px 30px;
    text-align: left;
    width: 100%;                /* fill up the entire div */
}
 
#menu {
    float: left;
    border: 1px solid #000;
    border-bottom: none;        /* avoid a double border */
    clear: both;                /* clear:both makes sure the content div doesn't float next to this one but stays under it */
    width:100%;
    height:20px;
    padding: 0 30px;
    background-color: #FFF;
    text-align: left;
    font-size: 85%;
}
 
#menu a:hover {
    background-color: #009FC1;
}
 
#userbar {
    background-color: #fff;
    float: right;
    width: 250px;
}
 
#footer {
    clear: both;
}
 
/* begin table styles */
table {
    border-collapse: collapse;
    width: 100%;
}
 
table a {
    color: #000;
}
 
table a:hover {
    color:#373737;
    text-decoration: none;
}
 
th {
    background-color: #B40E1F;
    color: #F0F0F0;
}
 
td {
    padding: 5px;
}
 
/* Begin font styles */
h1, #footer {
    font-family: Arial;
    color: #F1F3F1;
}
 
h3 {margin: 0; padding: 0;}
 
/* Menu styles */
.item {
    background-color: #00728B;
    border: 1px solid #032472;
    color: #FFF;
    font-family: Arial;
    padding: 3px;
    text-decoration: none;
}
 
.leftpart {
    width: 70%;
}
 
.rightpart {
    width: 30%;
}
 
.small {
    font-size: 75%;
    color: #373737;
}
#footer {
    font-size: 65%;
    padding: 3px 0 0 0;
}
 
.topic-post {
    height: 100px;
    overflow: auto;
}
 
.post-content {
    padding: 30px;
}
 
textarea {
    width: 500px;
    height: 200px;
}
```

## Step 3: Getting Ready for Action

Before we can read anything from our database, we need a connection. That's what connect.php is for. We'll include it in every file we are going to create.

```php
<?php
//connect.php
$server = 'localhost';
$username   = 'usernamehere';
$password   = 'passwordhere';
$database   = 'databasenamehere';
 
if(!mysql_connect($server, $username,  $password))
{
    exit('Error: could not establish database connection');
}
if(!mysql_select_db($database)
{
    exit('Error: could not select the database');
}
?>
```

Simply replace the default values of the variables at the top of the page with your own date, save the file and you're good to go!

## Step 4: Displaying the Forum Overview

Since we're just started with some basic techniques, we're going to make a simplified version of the forum overview for now.

```php
<?php
//create_cat.php
include 'connect.php';
include 'header.php';
         
echo '<tr>';
    echo '<td class="leftpart">';
        echo '<h3><a href="category.php?id=">Category name</a></h3> Category description goes here';
    echo '</td>';
    echo '<td class="rightpart">';                
            echo '<a href="topic.php?id=">Topic subject</a> at 10-10';
    echo '</td>';
echo '</tr>';
include 'footer.php';
?>
```

There you have it: a nice and clean overview. We'll be updating this page throughout the tutorial so that it becomes more like the end result, step by step!

## Step 5: Signing up a User

Let's start by making a simple HTML form so that a new user can register.


A PHP page is needed to process the form. We're going to use a \$\_SERVER variable. The \$\_SERVER variable is an array with values that are automatically set with each request. One of the values of the \$\_SERVER array is 'REQUEST_METHOD'. When a page is requested with GET, this variable will hold the value 'GET'. When a page is requested via POST, it will hold the value 'POST'. We can use this value to check if a form has been posted. See the signup.php page below.

```php
<?php
//signup.php
include 'connect.php';
include 'header.php';
 
echo '<h3>Sign up</h3>';
 
if($_SERVER['REQUEST_METHOD'] != 'POST')
{
    /*the form hasn't been posted yet, display it
      note that the action="" will cause the form to post to the same page it is on */
    echo '<form method="post" action="">
        Username: <input type="text" name="user_name" />
        Password: <input type="password" name="user_pass">
        Password again: <input type="password" name="user_pass_check">
        E-mail: <input type="email" name="user_email">
        <input type="submit" value="Add category" />
     </form>';
}
else
{
    /* so, the form has been posted, we'll process the data in three steps:
        1.  Check the data
        2.  Let the user refill the wrong fields (if necessary)
        3.  Save the data 
    */
    $errors = array(); /* declare the array for later use */
     
    if(isset($_POST['user_name']))
    {
        //the user name exists
        if(!ctype_alnum($_POST['user_name']))
        {
            $errors[] = 'The username can only contain letters and digits.';
        }
        if(strlen($_POST['user_name']) > 30)
        {
            $errors[] = 'The username cannot be longer than 30 characters.';
        }
    }
    else
    {
        $errors[] = 'The username field must not be empty.';
    }
     
     
    if(isset($_POST['user_pass']))
    {
        if($_POST['user_pass'] != $_POST['user_pass_check'])
        {
            $errors[] = 'The two passwords did not match.';
        }
    }
    else
    {
        $errors[] = 'The password field cannot be empty.';
    }
     
    if(!empty($errors)) /*check for an empty array, if there are errors, they're in this array (note the ! operator)*/
    {
        echo 'Uh-oh.. a couple of fields are not filled in correctly..';
        echo '<ul>';
        foreach($errors as $key => $value) /* walk through the array so all the errors get displayed */
        {
            echo '<li>' . $value . '</li>'; /* this generates a nice error list */
        }
        echo '</ul>';
    }
    else
    {
        //the form has been posted without, so save it
        //notice the use of mysql_real_escape_string, keep everything safe!
        //also notice the sha1 function which hashes the password
        $sql = "INSERT INTO
                    users(user_name, user_pass, user_email ,user_date, user_level)
                VALUES('" . mysql_real_escape_string($_POST['user_name']) . "',
                       '" . sha1($_POST['user_pass']) . "',
                       '" . mysql_real_escape_string($_POST['user_email']) . "',
                        NOW(),
                        0)";
                         
        $result = mysql_query($sql);
        if(!$result)
        {
            //something went wrong, display the error
            echo 'Something went wrong while registering. Please try again later.';
            //echo mysql_error(); //debugging purposes, uncomment when needed
        }
        else
        {
            echo 'Successfully registered. You can now <a href="signin.php">sign in</a> and start posting! :-)';
        }
    }
}

include 'footer.php';

?>
```

A lot of explanation is in the comments I made in the file, so be sure to check them out. The processing of the data takes place in three parts:

Validating the data
If the data is not valid, show the form again
If the data is valid, save the record in the database
The PHP part is quite self-explanatory. The SQL-query however probably needs a little more explanation.

```
INSERT INTO
       users(user_name, user_pass, user_email ,user_date, user_level)
VALUES('" . mysql_real_escape_string($_POST['user_name']) . "',
       '" . sha1($_POST['user_pass']) . "',
       '" . mysql_real_escape_string($_POST['user_email']) . "',
       NOW(),   
       0);
```

On line 1 we have the INSERT INTO statement which speaks for itself. The table name is specified on the second line. The words between the brackets represent the columns in which we want to insert the data. The VALUES statement tells the database we're done declaring column names and it's time to specify the values. There is something new here: mysql_real_escape_string. The function escapes special characters in an unescaped string , so that it is safe to place it in a query. This function MUST always be used, with very few exceptions. There are too many scripts that don't use it and can be hacked real easy. Don't take the risk, use mysql_real_escape_string().

"Never insert a plain password as-is. You MUST always encrypt it."
Also, you can see that the function sha1() is used to encrypt the user's password. This is also a very important thing to remember. Never insert a plain password as-is. You MUST always encrypt it. Imagine a hacker who somehow manages to get access to your database. If he sees all the plain-text passwords he could log into any (admin) account he wants. If the password columns contain sha1 strings he has to crack them first which is almost impossible.

Note: it's also possible to use md5(), I always use sha1() because benchmarks have proved it's a tiny bit faster, not much though. You can replace sha1 with md5 if you like.

If the signup process was successful, you should see something like this:


Try refreshing your phpMyAdmin screen, a new record should be visible in the users table.

## Step 6: Adding Authentication and User Levels

An important aspect of a forum is the difference between regular users and admins/moderators. Since this is a small forum and adding features like adding new moderators and stuff would take way too much time, we'll focus on the login process and create some admin features like creating new categories and closing a thread.

Now that you've completed the previous step, we're going to make your freshly created account an admin account. In phpMyAdmin, click on the users table, and then 'Browse'. Your account will probably pop up right away. Click the edit icon and change the value of the user_level field from 0 to 1. That's it for now. You won't notice any difference in our application immediately, but when we've added the admin features a normal account and your account will have different capabilities.

The sign-in process works the following way:

A visitor enters user data and submits the form
If the username and password are correct, we can start a session
If the username and password are incorrect, we show the form again with a message

The signin.php file is below. Don't think I'm not explaining what I'm doing, but check out the comments in the file. It's much easier to understand that way.

```php
<?php
//signin.php
include 'connect.php';
include 'header.php';
 
echo '<h3>Sign in</h3>';
 
//first, check if the user is already signed in. If that is the case, there is no need to display this page
if(isset($_SESSION['signed_in']) && $_SESSION['signed_in'] == true)
{
    echo 'You are already signed in, you can <a href="signout.php">sign out</a> if you want.';
}
else
{
    if($_SERVER['REQUEST_METHOD'] != 'POST')
    {
        /*the form hasn't been posted yet, display it
          note that the action="" will cause the form to post to the same page it is on */
        echo '<form method="post" action="">
            Username: <input type="text" name="user_name" />
            Password: <input type="password" name="user_pass">
            <input type="submit" value="Sign in" />
         </form>';
    }
    else
    {
        /* so, the form has been posted, we'll process the data in three steps:
            1.  Check the data
            2.  Let the user refill the wrong fields (if necessary)
            3.  Varify if the data is correct and return the correct response
        */
        $errors = array(); /* declare the array for later use */
         
        if(!isset($_POST['user_name']))
        {
            $errors[] = 'The username field must not be empty.';
        }
         
        if(!isset($_POST['user_pass']))
        {
            $errors[] = 'The password field must not be empty.';
        }
         
        if(!empty($errors)) /*check for an empty array, if there are errors, they're in this array (note the ! operator)*/
        {
            echo 'Uh-oh.. a couple of fields are not filled in correctly..';
            echo '<ul>';
            foreach($errors as $key => $value) /* walk through the array so all the errors get displayed */
            {
                echo '<li>' . $value . '</li>'; /* this generates a nice error list */
            }
            echo '</ul>';
        }
        else
        {
            //the form has been posted without errors, so save it
            //notice the use of mysql_real_escape_string, keep everything safe!
            //also notice the sha1 function which hashes the password
            $sql = "SELECT 
                        user_id,
                        user_name,
                        user_level
                    FROM
                        users
                    WHERE
                        user_name = '" . mysql_real_escape_string($_POST['user_name']) . "'
                    AND
                        user_pass = '" . sha1($_POST['user_pass']) . "'";
                         
            $result = mysql_query($sql);
            if(!$result)
            {
                //something went wrong, display the error
                echo 'Something went wrong while signing in. Please try again later.';
                //echo mysql_error(); //debugging purposes, uncomment when needed
            }
            else
            {
                //the query was successfully executed, there are 2 possibilities
                //1. the query returned data, the user can be signed in
                //2. the query returned an empty result set, the credentials were wrong
                if(mysql_num_rows($result) == 0)
                {
                    echo 'You have supplied a wrong user/password combination. Please try again.';
                }
                else
                {
                    //set the $_SESSION['signed_in'] variable to TRUE
                    $_SESSION['signed_in'] = true;
                     
                    //we also put the user_id and user_name values in the $_SESSION, so we can use it at various pages
                    while($row = mysql_fetch_assoc($result))
                    {
                        $_SESSION['user_id']    = $row['user_id'];
                        $_SESSION['user_name']  = $row['user_name'];
                        $_SESSION['user_level'] = $row['user_level'];
                    }
                     
                    echo 'Welcome, ' . $_SESSION['user_name'] . '. <a href="index.php">Proceed to the forum overview</a>.';
                }
            }
        }
    }
}
 
include 'footer.php';
?>
```

This is the query that's in the signin.php file:

```
SELECT
    user_id,
    user_name,
    user_level
FROM
    users
WHERE
    user_name = '" . mysql_real_escape_string($_POST['user_name']) . "'
AND
    user_pass = '" . sha1($_POST['user_pass'])
```

It's obvious we need a check to tell if the supplied credentials belong to an existing user. A lot of scripts retrieve the password from the database and compare it using PHP. If we do this directly via SQL the password will be stored in the database once during registration and never leave it again. This is safer, because all the real action happens in the database layer and not in our application.

If the user is signed in successfully, we're doing a few things:

```php
<?php
//set the $_SESSION['signed_in'] variable to TRUE
$_SESSION['signed_in'] = true;                  
//we also put the user_id and user_name values in the $_SESSION, so we can use it at various pages
while($row = mysql_fetch_assoc($result))
{
    $_SESSION['user_id'] = $row['user_id'];
    $_SESSION['user_name'] = $row['user_name']; 
}
?>
```

First, we set the 'signed_in' \$\_SESSION var to true, so we can use it on other pages to make sure the user is signed in. We also put the username and user id in the \$\_SESSION variable for usage on a different page. Finally, we display a link to the forum overview so the user can get started right away.

Of course signing in requires another function, signing out! The sign-out process is actually a lot easier than the sign-in process. Because all the information about the user is stored in \$\_SESSION variables, all we have to do is unset them and display a message.

Now that we've set the \$\_SESSION variables, we can determine if someone is signed in. Let's make a last simple change to header.php:

Replace:

```html
<div id="userbar">Hello Example. Not you? Log out.</div>
```

With:

```php
<?php
<div id="userbar">
    if($_SESSION['signed_in'])
    {
        echo 'Hello' . $_SESSION['user_name'] . '. Not you? <a href="signout.php">Sign out</a>';
    }
    else
    {
        echo '<a href="signin.php">Sign in</a> or <a href="sign up">create an account</a>.';
    }
</div>
```

If a user is signed in, he will see his or her name displayed on the front page with a link to the signout page. Our authentication is done! By now our forum should look like this:

## Step 7: Creating a Category

We want to create categories so let's start with making a form.

```html
<form method="post" action="">
    Category name: <input type="text" name="cat_name" />
    Category description: <textarea name="cat_description" /></textarea>
    <input type="submit" value="Add category" />
 </form>
 ```
 
This step looks a lot like Step 4 (Signing up a user'), so I'm not going to do an in-depth explanation here. If you followed all the steps you should be able to understand this somewhat quickly.

```php
<?php
//create_cat.php
include 'connect.php';
 
if($_SERVER['REQUEST_METHOD'] != 'POST')
{
    //the form hasn't been posted yet, display it
    echo '<form method='post' action=''>
        Category name: <input type='text' name='cat_name' />
        Category description: <textarea name='cat_description' /></textarea>
        <input type='submit' value='Add category' />
     </form>';
}
else
{
    //the form has been posted, so save it
    $sql = ìINSERT INTO categories(cat_name, cat_description)
       VALUES('' . mysql_real_escape_string($_POST['cat_name']) . ì',
             '' . mysql_real_escape_string($_POST['cat_description']) . ì')';
    $result = mysql_query($sql);
    if(!$result)
    {
        //something went wrong, display the error
        echo 'Error' . mysql_error();
    }
    else
    {
        echo 'New category successfully added.';
    }
}
?>
```

As you can see, we've started the script with the \$\_SERVER check, after checking if the user has admin rights, which is required for creating a category. The form gets displayed if it hasn't been submitted already. If it has, the values are saved. Once again, a SQL query is prepared and then executed.

## Step 8: Adding Categories to index.php

We've created some categories, so now we're able to display them on the front page. Let's add the following query to the content area of index.php.

```
SELECT
    categories.cat_id,
    categories.cat_name,
    categories.cat_description,
FROM
    categories
```

This query selects all categories and their names and descriptions from the categories table. We only need a bit of PHP to display the results. If we add that part just like we did in the previous steps, the code will look like this.

```
<?php
//create_cat.php
include 'connect.php';
include 'header.php';
 
$sql = "SELECT
            cat_id,
            cat_name,
            cat_description,
        FROM
            categories";
 
$result = mysql_query($sql);
 
if(!$result)
{
    echo 'The categories could not be displayed, please try again later.';
}
else
{
    if(mysql_num_rows($result) == 0)
    {
        echo 'No categories defined yet.';
    }
    else
    {
        //prepare the table
        echo '<table border="1">
              <tr>
                <th>Category</th>
                <th>Last topic</th>
              </tr>'; 
             
        while($row = mysql_fetch_assoc($result))
        {               
            echo '<tr>';
                echo '<td class="leftpart">';
                    echo '<h3><a href="category.php?id">' . $row['cat_name'] . '</a></h3>' . $row['cat_description'];
                echo '</td>';
                echo '<td class="rightpart">';
                            echo '<a href="topic.php?id=">Topic subject</a> at 10-10';
                echo '</td>';
            echo '</tr>';
        }
    }
}
 
include 'footer.php';
?>
```

Notice how we're using the cat_id to create links to category.php. All the links to this page will look like this: category.php?cat\_id=x, where x can be any numeric value. This may be new to you. We can check the url with PHP for \$\_GET values. For example, we have this link:

1
category.php?cat_id=23

The statement echo \$\_GET[ëcat_id'];' will display '23'. In the next few steps we'll use this value to retrieve the topics when viewing a single category, but topics can't be viewed if we haven't created them yet. So let's create some topics!

## Step 9: Creating a Topic

In this step, we're combining the techniques we learned in the previous steps. We're checking if a user is signed in, we'll use an input query to create the topic and create some basic HTML forms.

The structure of create_topic.php can hardly be explained in a list or something, so I rewrote it in pseudo-code.

```php
<?php
if(user is signed in)
{
    //the user is not signed in
}
else
{
    //the user is signed in
    if(form has not been posted)
    {   
        //show form
    }
    else
    {
        //process form
    }
}
?>
```

Here's the real code of this part of our forum, check the explanations below the code to see what it's doing.

```php
<?php
//create_cat.php
include 'connect.php';
include 'header.php';
 
echo '<h2>Create a topic</h2>';
if($_SESSION['signed_in'] == false)
{
    //the user is not signed in
    echo 'Sorry, you have to be <a href="/forum/signin.php">signed in</a> to create a topic.';
}
else
{
    //the user is signed in
    if($_SERVER['REQUEST_METHOD'] != 'POST')
    {   
        //the form hasn't been posted yet, display it
        //retrieve the categories from the database for use in the dropdown
        $sql = "SELECT
                    cat_id,
                    cat_name,
                    cat_description
                FROM
                    categories";
         
        $result = mysql_query($sql);
         
        if(!$result)
        {
            //the query failed, uh-oh :-(
            echo 'Error while selecting from database. Please try again later.';
        }
        else
        {
            if(mysql_num_rows($result) == 0)
            {
                //there are no categories, so a topic can't be posted
                if($_SESSION['user_level'] == 1)
                {
                    echo 'You have not created categories yet.';
                }
                else
                {
                    echo 'Before you can post a topic, you must wait for an admin to create some categories.';
                }
            }
            else
            {
         
                echo '<form method="post" action="">
                    Subject: <input type="text" name="topic_subject" />
                    Category:'; 
                 
                echo '<select name="topic_cat">';
                    while($row = mysql_fetch_assoc($result))
                    {
                        echo '<option value="' . $row['cat_id'] . '">' . $row['cat_name'] . '</option>';
                    }
                echo '</select>'; 
                     
                echo 'Message: <textarea name="post_content" /></textarea>
                    <input type="submit" value="Create topic" />
                 </form>';
            }
        }
    }
    else
    {
        //start the transaction
        $query  = "BEGIN WORK;";
        $result = mysql_query($query);
         
        if(!$result)
        {
            //Damn! the query failed, quit
            echo 'An error occured while creating your topic. Please try again later.';
        }
        else
        {
     
            //the form has been posted, so save it
            //insert the topic into the topics table first, then we'll save the post into the posts table
            $sql = "INSERT INTO 
                        topics(topic_subject,
                               topic_date,
                               topic_cat,
                               topic_by)
                   VALUES('" . mysql_real_escape_string($_POST['topic_subject']) . "',
                               NOW(),
                               " . mysql_real_escape_string($_POST['topic_cat']) . ",
                               " . $_SESSION['user_id'] . "
                               )";
                      
            $result = mysql_query($sql);
            if(!$result)
            {
                //something went wrong, display the error
                echo 'An error occured while inserting your data. Please try again later.' . mysql_error();
                $sql = "ROLLBACK;";
                $result = mysql_query($sql);
            }
            else
            {
                //the first query worked, now start the second, posts query
                //retrieve the id of the freshly created topic for usage in the posts query
                $topicid = mysql_insert_id();
                 
                $sql = "INSERT INTO
                            posts(post_content,
                                  post_date,
                                  post_topic,
                                  post_by)
                        VALUES
                            ('" . mysql_real_escape_string($_POST['post_content']) . "',
                                  NOW(),
                                  " . $topicid . ",
                                  " . $_SESSION['user_id'] . "
                            )";
                $result = mysql_query($sql);
                 
                if(!$result)
                {
                    //something went wrong, display the error
                    echo 'An error occured while inserting your post. Please try again later.' . mysql_error();
                    $sql = "ROLLBACK;";
                    $result = mysql_query($sql);
                }
                else
                {
                    $sql = "COMMIT;";
                    $result = mysql_query($sql);
                     
                    //after a lot of work, the query succeeded!
                    echo 'You have successfully created <a href="topic.php?id='. $topicid . '">your new topic</a>.';
                }
            }
        }
    }
}
 
include 'footer.php';
?>
```

I'll discuss this page in two parts, showing the form and processing the form.

Showing the form
We're starting with a simple HTML form. There is actually something special here, because we use a dropdown. This dropdown is filled with data from the database, using this query:

```
SELECT
    cat_id,
    cat_name,
    cat_description
FROM
    categories
```

That's the only potentially confusing part here; it's quite a piece of code, as you can see when looking at the create_topic.php file at the bottom of this step.

Processing the form

The process of saving the topic consists of two parts: saving the topic in the topics table and saving the first post in the posts table. This requires something quite advanced that goes a bit beyond the scope of this tutorial. It's called a transaction, which basically means that we start by executing the start command and then rollback when there are database errors and commit when everything went well. More about transactions.

```php
<?php
//start the transaction
$query  = "BEGIN WORK;";
$result = mysql_query($query);
//stop the transaction
$sql = "ROLLBACK;";
$result = mysql_query($sql);
//commit the transaction
$sql = "COMMIT;";
$result = mysql_query($sql);
?>
```

The first query being used to save the data is the topic creation query, which looks like this:

```
INSERT INTO
    topics(topic_subject,
               topic_date,
               topic_cat,
               topic_by)
VALUES('" . mysql_real_escape_string($_POST['topic_subject']) . "',
       NOW(),
       " . mysql_real_escape_string($_POST['topic_cat']) . ",
       " . $_SESSION['user_id'] . ")
```

At first the fields are defined, then the values to be inserted. We've seen the first one before, it's just a string which is made safe by using mysql_real_escape_string(). The second value, NOW(), is a SQL function for the current time. The third value, however, is a value we haven't seen before. It refers to a (valid) id of a category. The last value refers to an (existing) user_id which is, in this case, the value of \$\_SESSION[ëuser_id']. This variable was declared during the sign in process.

If the query executed without errors we proceed to the second query. Remember we are still doing a transaction here. If we would've got errors we would have used the ROLLBACK command.

```
INSERT INTO
        posts(post_content,
        post_date,
        post_topic,
        post_by)
VALUES
        ('" . mysql_real_escape_string($_POST['post_content']) . "',
         NOW(),
         " . $topicid . ",
         " . $_SESSION['user_id'] . ")
```

The first thing we do in this code is use mysql_insert_id() to retrieve the latest generated id from the topic_id field in the topics table. As you may remember from the first steps of this tutorial, the id is generated in the database using auto_increment.

Then the post is inserted into the posts table. This query looks a lot like the topics query. The only difference is that this post refers to the topic and the topic referred to a category. From the start, we decided to create a good data model and here is the result: a nice hierarchical structure.


## Step 10: Category View

We're going to make an overview page for a single category. We've just created a category, it would be handy to be able to view all the topics in it. First, create a page called category.php.

A short list of the things we need:

Needed for displaying the category
cat_name
cat_description
Needed for displaying all the topics

topic_id
topic_subject
topic_date
topic_cat

Let's create the two SQL queries that retrieve exactly this data from the database.

```
SELECT
    cat_id,
    cat_name,
    cat_description
FROM
    categories
WHERE
    cat_id = " . mysql_real_escape_string($_GET['id'])
```

The query above selects all the categories from the database.

```
SELECT 
    topic_id,
    topic_subject,
    topic_date,
    topic_cat
FROM
    topics
WHERE
    topic_cat = " . mysql_real_escape_string($_GET['id'])
```

The query above is executed in the while loop in which we echo the categories. By doing it this way, we'll see all the categories and the latest topic for each of them.
The complete code of category.php will be the following:

```php
<?php
//create_cat.php
include 'connect.php';
include 'header.php';
 
//first select the category based on $_GET['cat_id']
$sql = "SELECT
            cat_id,
            cat_name,
            cat_description
        FROM
            categories
        WHERE
            cat_id = " . mysql_real_escape_string($_GET['id']);
 
$result = mysql_query($sql);
 
if(!$result)
{
    echo 'The category could not be displayed, please try again later.' . mysql_error();
}
else
{
    if(mysql_num_rows($result) == 0)
    {
        echo 'This category does not exist.';
    }
    else
    {
        //display category data
        while($row = mysql_fetch_assoc($result))
        {
            echo '<h2>Topics in ′' . $row['cat_name'] . '′ category</h2>';
        }
     
        //do a query for the topics
        $sql = "SELECT  
                    topic_id,
                    topic_subject,
                    topic_date,
                    topic_cat
                FROM
                    topics
                WHERE
                    topic_cat = " . mysql_real_escape_string($_GET['id']);
         
        $result = mysql_query($sql);
         
        if(!$result)
        {
            echo 'The topics could not be displayed, please try again later.';
        }
        else
        {
            if(mysql_num_rows($result) == 0)
            {
                echo 'There are no topics in this category yet.';
            }
            else
            {
                //prepare the table
                echo '<table border="1">
                      <tr>
                        <th>Topic</th>
                        <th>Created at</th>
                      </tr>'; 
                     
                while($row = mysql_fetch_assoc($result))
                {               
                    echo '<tr>';
                        echo '<td class="leftpart">';
                            echo '<h3><a href="topic.php?id=' . $row['topic_id'] . '">' . $row['topic_subject'] . '</a><h3>';
                        echo '</td>';
                        echo '<td class="rightpart">';
                            echo date('d-m-Y', strtotime($row['topic_date']));
                        echo '</td>';
                    echo '</tr>';
                }
            }
        }
    }
}
 
include 'footer.php';
?>
```

And here is the final result of our categories page:

## Step 11: Topic View

The SQL queries in this step are complicated ones. The PHP-part is all stuff that you've seen before. Let's take a look at the queries. The first one retrieves basic information about the topic:

```
SELECT
    topic_id,
    topic_subject
FROM
    topics
WHERE
    topics.topic_id = " . mysql_real_escape_string($_GET['id'])
```

This information is displayed in the head of the table we will use to display all the data. Next, we retrieve all the posts in this topic from the database. The following query gives us exactly what we need:

```
SELECT
    posts.post_topic,
    posts.post_content,
    posts.post_date,
    posts.post_by,
    users.user_id,
    users.user_name
FROM
    posts
LEFT JOIN
    users
ON
    posts.post_by = users.user_id
WHERE
    posts.post_topic = " . mysql_real_escape_string($_GET['id'])
```

This time, we want information from the users and the posts table - so we use the LEFT JOIN again. The condition is: the user id should be the same as the post_by field. This way we can show the username of the user who replied at each post.

The final topic view looks like this:

## Step 12: Adding a Reply

Let's create the last missing part of this forum, the possibility to add a reply. We'll start by creating a form:

```html
<form method="post" action="reply.php?id=5">
    <textarea name="reply-content"></textarea>
    <input type="submit" value="Submit reply" />
</form>
```

The complete reply.php code looks like this.

```php
<?php
//create_cat.php
include 'connect.php';
include 'header.php';
 
if($_SERVER['REQUEST_METHOD'] != 'POST')
{
    //someone is calling the file directly, which we don't want
    echo 'This file cannot be called directly.';
}
else
{
    //check for sign in status
    if(!$_SESSION['signed_in'])
    {
        echo 'You must be signed in to post a reply.';
    }
    else
    {
        //a real user posted a real reply
        $sql = "INSERT INTO 
                    posts(post_content,
                          post_date,
                          post_topic,
                          post_by) 
                VALUES ('" . $_POST['reply-content'] . "',
                        NOW(),
                        " . mysql_real_escape_string($_GET['id']) . ",
                        " . $_SESSION['user_id'] . ")";
                         
        $result = mysql_query($sql);
                         
        if(!$result)
        {
            echo 'Your reply has not been saved, please try again later.';
        }
        else
        {
            echo 'Your reply has been saved, check out <a href="topic.php?id=' . htmlentities($_GET['id']) . '">the topic</a>.';
        }
    }
}
 
include 'footer.php';
?>
```

The comments in the code pretty much detail what's happening. We're checking for a real user and then inserting the post into the database.

## Finishing Up

Now that you've finished this tutorial, you should have a much better understanding of what it takes to build a forum. I hope my explanations were clear enough! Thanks again for reading.