# CleverDict

![simplicity](https://image.slidesharecdn.com/iotshifts20150911-151021225240-lva1-app6891/95/smart-citizens-populating-smart-cities-iotshifts-19-638.jpg?cb=1506979421)

## OVERVIEW

```CleverDict``` is a hybrid Python data class which allows both ```object.attribute``` and ```dictionary['key']``` notation to be used simultaneously and interchangeably.  It's particularly handy when your code is mainly object-orientated but you want a 'DRY' and extensible way to import data in json/dictionary format into your objects... or vice versa... without having to write extra code just to handle the translation.

The class also optionally triggers a ```._save()``` method (which you can adapt or overwrite) which it calls whenever an attribute or dictionary value is created or changed.  This is especially useful if you want your object's values to be automatically pickled, encoded, saved to a file or database, uploaded to the cloud etc. without having to slavishly call your update function after every single operation where attributes (might) change.


## INSTALLATION
No dependencies.  Very lightweight:

    pip install cleverdict

or to cover all bases...

    python -m pip install cleverdict --upgrade --user

## QUICKSTART

```CleverDict``` objects behave like normal Python dictionaries, but with the convenience of immediately offering read and write access to their data (keys and values) using the ```object.attribute``` syntax, which many people find easier to type and more intuitive to read and understand.

### 1. BASIC USE

    >>> from cleverdict import CleverDict
    >>> x = CleverDict({'total':6, 'usergroup': "Knights of Ni"})

    >>> x.total
    6
    >>> x['total']
    6
    >>> x.usergroup
    'Knights of Ni'
    >>> x['usergroup']
    'Knights of Ni'

### 2. KEYWORD ARGUMENTS
You can also supply keyword arguments like this:

    >>> x = CleverDict(created = "today", review = "tomorrow")

    >>> x.created
    'today'
    >>> x['review']
    'tomorrow'

Regardless of which syntax you use, new values are immediately available via both methods:

    >>> x['life'] = 42
    >>> x.life += 1
    >>> x['life']
    43

    >>> del x['life']
    >>> x.life
    # KeyError: 'life'

### 3. NORMALISATION
By default ```CleverDict``` attempts to normalise keys such as " " and "" and strings with characters not normally allowed in ```object.attribute``` names.  So for example ```1``` (integer) becomes ```"_1"``` (string):


    >>> x = CleverDict({1: "One"})

    >>> x._1
    'One'
    >>> x
    CleverDict({'_1':'One'})


You can toggle this behaviour Off and On (for all future operations on all current and future objects of the class) as follows:


    >>> CleverDict.normalise = False  # or: True
    >>> x = CleverDict({1: "One"})

    >>> hasattr(x, "_1")
    False
    >>> x
    CleverDict({1:'One'})
    >>> x[1]
    'One'

### 4. IMPORTING DATA FROM OTHER OBJECTS
You can import an existing object's data (but not its methods) directly using ```vars()```:

    x = CleverDict(vars(my_existing_object))

    >>> list(x.items())
    [('total', 6), ('usergroup', 'Knights of Ni'), ('life', 42)]

### 5. ENABLING THE AUTO-SAVE FUNCTION
You can set pretty much any function to run automatically whenever a ```CleverDict``` value is created or changed.  There's an example function in ```cleverict.test_cleverdict``` which demonstrates this:

    >>> from cleverdict.test_cleverdict import my_example_save_function
    >>> CleverDict._save = my_example_save_function

    >>> x = CleverDict({'total':6, 'usergroup': "Knights of Ni"})
    Notional save to database: .total = 6 <class 'int'>
    Notional save to database: .usergroup = Knights of Ni <class 'str'>

    >>> x.life = 42
    Notional save to database: .life = 42 <class 'int'>


The example function above also appends output to a file, which you might want for debugging, auditing,  further analysis etc.:

    >>> with open("example.log","r") as file:
    ...     log = file.read()

    >>> log.splitlines()
    ["Notional save to database: .age = 10 <class 'int'>",
    "Notional save to database: .total = 6 <class 'int'>",
    "Notional save to database: .usergroup = Knights of Ni <class 'str'>"]

**NB**: The ```._save()``` method is a *class* method, so changing ```CleverDict._save``` will apply the new ```._save()``` method to all previously created ```CleverDict``` objects as well as future ones.


### 6. CREATING YOUR OWN AUTO-SAVE FUNCTION
When writing your own ```._save()``` function, you'll need to specify three arguments as follows:


    >>> def your_function(self, name: str = "", value: any = ""):
    ...     print("Ni!")


* **self**: because we're dealing with objects and classes...
* **name**: a valid Python ```.attribute``` name preferably, otherwise you'll only be able to access it using ```dictionary['key']``` notation later on.
* **value**: anything

### 7. SETTING DIFFERENT AUTO-SAVE FUNCTIONS FOR DIFFERENT OBJECTS
If you want to specify different ```._save()``` behaviours for different objects, consider creating sublasses that inherit from ```CleverDict``` and set a different
```._save()``` function for each subclass e.g.:

    >>> class Type1(CleverDict): pass
    >>> Type1._save = my_save_function1

    >>> class Type2(CleverDict): pass
    >>> Type2._save = my_save_function2


### 8. CHANGING THE WAY CLEVERDICT OBJECTS ARE DISPLAYED
If you want to overwrite the ```__str__``` method with your own function, or delete it so ```print()``` and ```str()``` default to ```__repr__``` that's easy enough too:

    >>> print(x)
    .total = 6 <class 'int'>
    .usergroup = Knights of Ni <class 'str'>

    >>> def my__str__replacement(self):
    ...     return str(type(self))

    >>> setattr(CleverDict, "__str__", my__str__replacement)
    >>> print(x)
    <class 'cleverdict.cleverdict.CleverDict'>

    >>> delattr(CleverDict, "__str__")
    >>> print(x)
    CleverDict({'total':6, 'usergroup':'Knights of Ni'})

## CONTRIBUTING

We'd love to see Pull Requests (and relevant tests) from other contributors, particularly if you can help with the following:

1. It would be great if ```CleverDict``` behaviour could be easily 'grafted on' to existing classes using inheritance, without causing recursion or requiring a rewrite/overwrite of the original class.

    For example if it were as easy as:

    ```
    >>> class MyDatetime(datetime.datetime, CleverDict):
    ...     pass

    >>> mdt = MyDatetime.now()
    >>> mdt.hour
    4
    >>> mdt['hour']
    4
    ```


## CREDITS
```CleverDict``` was developed jointly by Peter Fison, Ruud van der Ham, Loic Domaigne, and Rik Huygen who met on the friendly and excellent Pythonista Cafe forum (www.pythonistacafe.com).  Peter got the ball rolling after noticing a super-convenient, but not fully-fledged feature in Pandas that allows you to (mostly) use ```object.attribute``` syntax or ```dictionary['key']``` syntax interchangeably. Ruud, Loic and Rik then started swapping ideas for a hybrid  dictionary/data class based on ```UserDict``` and the magic of ```__getattr__``` and ```__setattr__```, and ```CleverDict``` was born*.

>(\*) ```CleverDict``` was originally called ```attr_dict``` but several confusing flavours of this and ```AttrDict``` exist on PyPi and Github already.  Hopefully the new name raises a wry smile as well as being more memorable...
