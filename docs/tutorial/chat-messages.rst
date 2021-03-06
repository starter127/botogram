.. Copyright (c) 2015 Pietro Albini <pietro@pietroalbini.io>
   Released under the MIT license

~~~~~~~~~~~~~~~~~~~~~
Parsing chat messages
~~~~~~~~~~~~~~~~~~~~~

Sometimes you need some pieces of information in a conversation, but you
forgot about that *really* useful command. In this cases, allowing the bot to
parse the chat and automatically send informations can be quite handy.

Even if some advanced parsing is... advanced, you can introduce a few simple
parsers, which will improve your chat experience.

.. note::

   Before continuing with this chapter, be sure to disable privacy mode for
   your bot via the @botfather (use the ``/setprivacy`` command). If you don't
   disable this, your bot won't receive normal chat messages in group chats,
   but only commands and so on.

.. warning::

   Some users won't use your bot if they see that privacy mode is disabled,
   especially in corporates or critical groups, so enable only if it's *really*
   necessary.

================================
Search for words in the messages
================================

The simplest way to parse chat messages is to search for words in it. This
won't allow you to perform complex matching, but it can be quite handy anyways.

In this example we want the bot to send the botogram GitHub page when someone
says in chat the word "botogram". To do so, we'll create this function:

.. code-block:: python
   :emphasize-lines: 3,4,5

       chat.send("Hello world")

   @bot.message_contains("botogram")
   def send_botogram_link(chat, message):
       pass

The :py:meth:`botogram.Bot.message_contains` decorator will register the hook,
and when a message contains the word "botogram", the ``send_botogram_link``
function will be called with these parameters:

* The chat where the command was issued (an instance of either
  :py:class:`botogram.User` or :py:class:`botogram.GroupChat`, depending on
  where the command was issued)
* The representation of the sent message (an instance of
  :py:class:`botogram.Message`)

Now we can simply send the GitHub link, like how we previously did in the
":ref:`tutorial-hello-world`" chapter:

.. code-block:: python
   :emphasize-lines: 3

   @bot.message_contains("botogram")
   def send_botogram_link(chat, message):
      chat.send("https://github.com/pietroalbini/botogram")

Perfect, you can now try to execute the bot an send to it the word "botogram":
you should receive that link!

=======================================
Match messages with regular expressions
=======================================

Now let's create something more advanced and useful. If you use GitHub for your
projects, you may need to reference issues to your coworkers. Instead of
copy-pasting the link every time, you can configure the bot to send the link
for you everytime you type something like "username/repo#issue".

.. note::

   Regular expressions can become fairly complex, and they aren't the easiest
   thing in the programming world. You can learn how they work by reading the
   `Regular Expression HOWTO`_ in the Python documentation.

The regular expression we will be using is
``([a-zA-Z0-9\-]+)/([a-zA-Z0-9\-\._]+)#([0-9]+)``, so let's start creating the
function:

.. code-block:: python
   :emphasize-lines: 3,4,5

       chat.send("https://github.com/pietroalbini/botogram")

   @bot.message_matches(r'([a-zA-Z0-9\-]+)/([a-zA-Z0-9\-\._]+)#([0-9]+)')
   def github_links(chat, message, matches):
       pass

The :py:meth:`botogram.Bot.message_matches` decorator will register the hook,
and when a message matches that regular expression, the ``github_links``
function will be called with the parameters you saw in the above section, plus
the ``matches`` one, which contains things the regular exception matched, as a
tuple.

Before sending the link, we should check it actually exists. In order to do so,
we'll be using requests_, a Python module also required also by botogram itself.
Let's import it:

.. code-block:: python
   :emphasize-lines: 2

   import botogram
   import requests

So, now we can actually check if the URL exists:

.. code-block:: python
   :emphasize-lines: 3,4,5

   @bot.message_matches(r'([a-zA-Z0-9\-]+)/([a-zA-Z0-9\-\._]+)#([0-9]+)')
   def github_links(chat, message, matches):
       url = "https://github.com/{}/{}/issues/{}".format(*matches)
       if requests.head(url).status_code != 404:
           chat.send(url)

That code will check if the hypothetic issue URL exists (so the status code
isn't ``404``), and if the URL exists the code will send it to the chat. Then
Telegram will automatically show the preview to the user.

===========================
Bot's source code until now
===========================

.. code-block:: python

   import botogram
   import requests

   bot = botogram.create("YOUR-API-KEY")
   bot.about = "This bot is just the one from the botogram's tutorial"
   bot.owner = "@yourusername"

   @bot.command("hello")
   def hello_command(chat, message, args):
       """Say hello to the world!
       This command sends "Hello world" to the current chat
       """
       chat.send("Hello world")

   @bot.message_contains("botogram")
   def send_botogram_link(chat, message):
       chat.send("https://github.com/pietroalbini/botogram")

   @bot.message_matches(r'([a-zA-Z0-9\-]+)/([a-zA-Z0-9\-\._]+)#([0-9]+)')
   def github_links(chat, message, matches):
       url = "https://github.com/{}/{}/issues/{}".format(*matches)
       if requests.head(url).status_code != 404:
           chat.send(url)

   bot.run()

.. _Regular Expression HOWTO: https://docs.python.org/3/howto/regex.html
.. _requests: http://docs.python-requests.org
