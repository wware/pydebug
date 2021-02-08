# Python Debugging Ideas to Tinker With

A bunch of ideas for improving the debugging facilities on the system I
maintain at work, which is a large legacy Python 2.7 codebase.

1. Check out https://github.com/romanvm/python-web-pdb, especially for post
   mortems.

1. Log to database

1. Use sys.settrace to run legacy Python code, when you get to some chosen
   line, run some other piece of code and then resume.

1. MORE ASSERTS in the codebase, and ADD COMMENTS to the asserts, or at least
   pointers, identifying - what issue is involved - what people are involved -
   which JIRA ticket is involved - history and implications. Maybe all that
   stuff lives offline in some text file and the assert only has a pointer into
   that file? Like a UUID for each one, or something.

## Ways to maybe improve Web-PDB???

When you start your program it brings up a web server which does a long polling
hack so that your browser will wait until a debugging session begins. when the
program hits an exception it goes into that debugging session, and the browser
alerts you in some way either graphically or audibly.

That said, it's hard to argue with the simplicity of something like this.

    import web_pdb
    with web_pdb.catch_post_mortem():
        run_my_code()

## Log to database

* `https://docs.python.org/3/library/logging.handlers.html#sockethandler`
* `https://docs.pylonsproject.org/projects/pyramid-cookbook/en/latest/logging/sqlalchemy_logger.html`

Here is the scenario I envision. I'm collecting a bunch of data and storing it
in a database, then later retrieving it to compare with a different run. And
you get some discrepancy that you need to explain. So you put entries in this
database with enough information to track down the data moving around and
identify anything unexpected.

You want the option to use a Docker container as the database where you log
stuff, so that you can throw it away when you're done.

## More asserts, and more descriptive/informative asserts

* `https://ensure.readthedocs.io/en/latest/`

I like the idea of "literate assertions" and the ensure thing is interesting,
but I think its approach to the problem isn't quite what I'm looking for. I
want to talk about WHY something is done and what HISTORY is involved and what
RELATED topics are important. The ensure thing is more about making the body of
the assertion look like English language. My problem is not that Python needs
to be more readable. My problem is capturing OTHER context.

So let's try an example of some sort. The "literacy" piece is not about making
`x != 0` look more readable, it's about providing information to help somebody
understand why we care about the value of `x`.

    msg = """
    This is some long involved discussion about why {0} should not be zero.
    If it's zero then you should refer to JIRA ticket FOOBAR-1234 and look
    at Alice's April 1st 2020 comment about database indexes and timezones.
    """
    assert x != 0, msg.format(x)

## Can you continue in Python after catching an exception?

Like take some corrective action but otherwise proceed as if exception didn't
happen, and do this with code that has not been modified. Particularly it would
be cool if you could do this in a debugger postmortem session, like you hit an
assert, then do something in the debugger, and then continue running code
starting with the statement right after the assert.

It doesn't look like there is any way to do that in Python. Nothing like a
continuation or anything, where you could dig around a bit in the stacktrace,
find the right frame, set whatever variables, and tell the thing to re-commence.

But this wish-list item should definitely be feasible:

> sys.settrace ... get to some chosen line, run some ... code and then resume.
