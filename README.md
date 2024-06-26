# jsonQM

A simple tool that can make your API endpoint json queries more functional.

`pip install jsonQM`

## Quick Start

The example model explains how to set up a model and query that model.

*tests/ExampleModel.py*

```py
from jsonQM import *

# Make the model.

# Declare a model serializer
ms = ModelSerializer() # Each model must have its own serializer
class MyModel(Model):
    def __init__(self) -> None:
        # define the structure of the query model:
        self.model = {
            # we have a single query scope/section called functions
            "functions":{},
            "msg":"test" # non functional attributes also need to be included in the json query if you wish for them to be present in the response (see Example query 3 below)
        }
        # sync/add the model attribute functions to the model
        ms.sync(self)
        
    # Define attributes of the model (argument 2) along with the scope/section they are found in (argument 1)
    @ms.add(["functions"], "repeater")  # This attribute key is "repeater" and can be found in the "functions" dict
    def repeater(self, arg:int):
        # when the attribute is queried it will run this code and return the value
        return ["repeat" for _ in range(arg)]

    # You can use anything as the attribute key that is a valid python dictionary key.
    @ms.add(["functions"], 7)# Keep in mind that if used with json, you are limited to what is a valid json key.
    def number_7(self):
        return "abc"
    
    @ms.token()# this marks this attribute as the model's token
    @ms.add([], "token")
    def token(self, tok:str):
        #insert some logic to compare token to tokens in database
        return tok == "token" # The token function must return true for the token to be valid.
    

    @ms.requires_token()# this marks this attribute as requiring the token attribute to return true
    @ms.add([], "secret")
    def secret(self):
        return "My super secret message" # if the token is false this will return 'errot:token:invalid' if the token is missing from the query this will return 'error:token:missing'

#LOGIC
if __name__ == "__main__":
    # If you are experiencing bugs related to concurrency, you should reinstantiate your model per thread/concurrent task where it is being used.
    # note that instantiation may require many itterations depending on the ammount of attributes you've added and the length of their parent path.
    # (parent path is the first list argument in `ms.add(["some","dictionary","key","path"], "attribute name")`)
    model_instance = MyModel()

    # If you can avoid instantiating the model more than once for optimal performance.

    #Example query 1
    print(model_instance.get({
        "functions":{
            # programmed attribute values should be a list containing the function arguments.
            "repeater":[5],
            7:[]
        }
    }))

    # prints:
    # {'functions': {'repeater': ['repeat', 'repeat', 'repeat', 'repeat', 'repeat'], 7: 'abc'}}

    #Example query 2
    print(model_instance.get({
        "functions":{
            # The model will only run/return attributes which have been specified
            "repeater":[5]
        }
    }))

    # prints:
    # {'functions': {'repeater': ['repeat', 'repeat', 'repeat', 'repeat', 'repeat']}}

    #Example query 3
    print(model_instance.get({
        "token":["token"],
        "secret":[]
    }))

    # prints:
    # {'token': True, 'secret': 'My super secret message'}

    #Example query 3
    print(model_instance.get({
        "token":["abc"],
        "secret":[]
    }))

    # prints:
    # {'token': False, 'secret': 'error:token:invalid'}

    #Example query 3
    print(model_instance.get({
        "secret":[]
    }))

    # prints:
    # {'secret': 'error:token:missing'}

    #Example query 3
    print(model_instance.get({
        "msg":1
    }))

    # prints:
    # {'msg': 'test'}
```
