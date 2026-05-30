# Setting up a linter and tests for your code
In this lab we will set up pylint and pytest so we can apply them to our code henceforth. We'll set up a job in our makefile so we can cycle fast.

Goals for this lab:
* Have pylint running on any code we point it to
* Have pytest running with some basic test
* Create a makefile task to automate the running of the tests/linter
* Have pytest augmented with os tests and other environment tests


# 1 Installing pylint and setting up config
You should have your virtual environment activated for this part.
* Add pylint to your requirements.txt.  (I think we already did previously, but add if
you don't see it add it now)
* Go ahead and run an update to make sure the requirements.txt is pip installed
* If you now activate your virtual environment you should have the pylint command
    - `pylint` should lead to -> `No files to lint: exiting.`

## 1.1 Setting up pylint config
You can find the documentation for pylint here -> 
[pylint readthedocsLinks](https://pylint.readthedocs.io/en/latest/tutorial.html).

You should be able to just run it directly on the new file, for example 
`pylint bin/clean_ids.py`. If you see a list of errors including
things like indentation, then it's working!

From the root of your repository, (in the same directory as
your makefile), run `pylint --generate-rcfile >> pylintrc`. This creates a config
file that will be read by pylint (it will look for this in your current
directory first, then in HOME etc).

If you run the pylint command again on your
file it should now be using the config file.  To test that your config is
actually being used, open the pylintrc file and search for `indent-string='    '`.
That should have 4 spaces in the string on the right of the = sign. Change that
to two spaces and run again. You should see a change in the error log depending on
the number of spaces.  Of course, this is not what we want, so set it back to 4 spaces,
we just wanted to be sure pylint is using it.

Make a new job in your makefile, `lint:` so you can
just re-execute that to lint your file. Remember to activate the virtual
environment first, just like we did before for the tests.  `make lint` should now
run the linter on your code base.

Verify the linter works, but **DO NOT FIX YOUR SCRIPT YET**.  Wait for tests to be in place.


# 2 Setting up pytest
We will use pytest to test our clean_ids.py script we created earlier.
* I am assuming you placed the script in a directory called `bin/`


Similarly as we did with pylint:
* Add pytest to your requirements.txt file.  Then execute `make update`.
* Create a `tests` directory and in that folder create a `test_<your module name here>.py`.
* Create a new file in your root directory called `pytest.ini`.  This allows your module to be imported from the root without having to force python to look for the `bin/` directory.
```
[pytest]
pythonpath = .
```
* Now import your module giving the path from the root of the repository, for example `import bin.clean_ids`
* Pytest will look for functions beginning with `test_`.  Your initial file should look like this:
```python
import sys
import io
import pytest
from bin.clean_ids import main

def test_script_execution(monkeypatch, capsys):
    # 1. Simulate the standard input data
    # We use io.StringIO to make a string act like a readable stream/file
    fake_input = io.StringIO("kcFsuxaJ1es\nasd123\n")
    monkeypatch.setattr(sys, "stdin", fake_input)

    # 2. Run the script's main logic
    main()

    # 3. Capture the printed output
    captured = capsys.readouterr()
    
    # 4. Assert that the data was modified correctly
    assert captured.out == "kcFsuxaJ1es\n"
```
* If all went well, you should now be able to run `pytest -vv tests` and see your seed test run.
* Add a make job called `test:` so you can simply run `make test`
* Finally, link the linter job you had before so `make test` runs the linter first.

## A note on the test code:
Typically you import a module that has functions, classes etc.  Testing is straight forward.
You import a module, say `import mymodule` and then inside the `test_` functions you use
the modules functions.  For example `assert mymodule.add(2,2) == 4`

In our case, our module has the added complication that we're using stdin and stdout.
In order to test this we need to do a few things:
1. Wrap your script content so it runs inside a function called `main()`, see step 2 in the sample test.
2. We `import io` to give us access to create a 'stream' object.  Then we use pytest's monkeypatch to change the stream to pretend it's console input, (last line in step 1)
3. We catch any error strings in step 3
4. Finally, we capture what the script would have printed to the console in step 4.

## Add additional tests:
This sample test checks one id and one bad line.  Now it's brainstorm time... possible additions...
* good id, bad line, good id
* bad line(s)
* test what constitutes a good id, 10 chars? 12 chars?
* what if it throws an error, do you want to add a quick test for that?



# 3 The goal: Using linter, tests and github in parallel to keep your coding `on rails`
Now that you have a linter and a place to test your code you have a framework to run tests as you modify your code.

## Getting the code up to snuff
After running the test/lint jobs, at the bottom of the pylint output you should see something like

```
------------------------------------------------------------------ Your code has
been rated at 2.86/10 (previous run: 3.57/10, -0.71)
```

Now cycle through.  Look at the output and if it's not 10/10 go through the errors.
You can look up the error codes here: [PYLINT ERROR CODES](https://pylint.readthedocs.io/en/latest/user_guide/messages/messages_overview.html).
Fix the errors one by one until you get as close to 10/10 as possible.
That may not be possible in some cases, but try your best and bring up error in Teams
or bring it up in office hours and we'll tackle them case by case.

# 4 Additional tests for environment
The reading should provide a good overview of how to add the following types of tests:

* What OS is the code running on?  It should check it's running on ubuntu,
* What version of python is running in the environment.
* Add a test that is expected to fail using a decorator.
* Add a test that is expected to be skipped (useful when you don't want to forget to add a test but the feature isn't ready)
* Finally add a parametrized test.


# Goals for this lab: You'll know you're done with this lab when
* You have a running `make lint` job that runs on your code and returns a quality score.
    - Note that you probably won't be able to get a 10/10, I believe we'll need to fiddle with the config.  However, get as close as you can and we can talk as to what people found was blocking the 10.
* You have a running `make test` job that runs some very general tests.  We'll add more tests as we add more scripts.
* You have generic tests for OS, python version etc, aside from the ones that test the script.
* Your `make test` runs the linter first then the tests



Links to documentation:
* pylint: https://pylint.readthedocs.io/en/latest/user_guide/checkers/features.html in
pylint so you can make sense of the feedback.
* pytest: https://docs.pytest.org/en/stable/contents.html



## POINTS 10
You should have
* All your tests be GREEN, skipped or expected to fail
* Your linter should be as close to 10/10

