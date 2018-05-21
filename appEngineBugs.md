# Common App Engine Bugs and Problems

Here are some common bugs I ran into using App Engine

## Another transaction by user <USER> is already in progress for app

    Error 409: --- begin server output ---
    Another transaction by user <USER> is already in progress for app: <App ID>, version: 1. That user can undo the transaction with "appcfg rollback".
    --- end server output ---

**Problem:** While making a deploy, your or another User quit unexpectedly. Instead of dealing with this like a responsible adult, App Engine shits itself and refuses to play nice until you make a `rollback` command.

**Solution:** (assuming you're using Python) In you project directory, issue the following command:

    appcfg.py rollback .

Also, sometimes there can be in issue with the cookies that App Engine stores in your home directory. One way of dealing with this is by deleting them. 

    rm ~/.appcfg_cookies
    rm .appcfg_oauth2_tokens

Or you can take the simpler, less scorch-earthy approach, and just use the inbuilt `--no_cookies` flag, like so:

    appcfg.py update . --no_cookies 


## This application does not exist

    Error 404: --- begin server output ---
    This application does not exist (app_id=u'jetdebt').
    --- end server output ---

**Problem:** The value you have in `app.yaml` for `application` is different to the identifier of your app. They must be the same. 

**Solution:** Change the value for `application` to be identical to your application identifier, which you can find in your App Engine console. 

1. Go to [https://appengine.google.com/](https://appengine.google.com/)
2. Click on the link to your app
3. In the sidebar, under `Administration`, click on `Application Settings`
4. Copy the value for `Application Identifier` to your clipboard
5. Go to your project's `app.yaml` file
6. Replace the value for `application` in `app.yaml` with the value you copied to your clipboard
7. Raise a glass of decent whiskey and toast to my good health

## WARNING util.py:126 new_request() takes at most 1 positional argument (6 given)

Basically, this seems to be a bullshit bug on AppEngine's side that they haven't figured out how to solve since 2013.

> I've confirmed the message is harmless so you can safely ignore it. We are working on a fix and should get one in to 1.8.4. answered Aug 12 '13 at 22:53 - coto

[http://stackoverflow.com/questions/18126157/appengine-warning-during-python-app-update](http://stackoverflow.com/questions/18126157/appengine-warning-during-python-app-update) 

As I come across more other common App Engine bugs, I'll update this post.

If you found this article helpful, please leave a `Thank you` comment below. 
 
