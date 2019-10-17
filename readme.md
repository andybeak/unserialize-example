# Example of insecure deserialization

This project is an example of insecure deserialization in PHP.

It contravenes the two golden rules:

1) Never unserialize using data that can be manipulated by a user
2) Always whitelist the class(es) that you intend to be deserialized

## Running the project

You can run the project with the following commands:

    composer install
    php -S localhost:8000
    
## Observing insecure deserialization

Let's imagine that a developer is trying to pass data between two systems.  They decide to serialize the data object and then make an HTTP call to an API endpoint.

This project is the receiving endpoint and they want to be able to perform operations on the `Pickle` object.

Visit this URL in your browser to view what we intend to happen:

    http://localhost:8000/?data=O:22:%22UnserializeDemo\Pickle%22:1:{s:4:%22name%22;s:4:%22Rick%22;}
    
It will output a message saying "Hi, Rick, the weather in London is absolutely lovely".

Visit this URL in your browser to effect an insecure deserialization attack:

    http://localhost:8000/?data=O:23:%22UnserializeDemo\Weather%22:2:{s:11:%22weatherData%22;s:27:%22cloudy%20with%20a%20chance%20of%20XSS%22;s:35:%22UnserializeDemo\WeathercachedData%22;s:17:%22absolutely%20lovely%22;}

### Nature of the attack
    
The attack is possible because the Weather class naively saves its data in the `__destruct()` method.  

Not only would this cause your cache never to be properly refreshed (and so is a bug), but it is also a security vulnerability because it lets attackers
write arbitrary content into the cache storage file.

### Mitigating the attack

Firstly, you should not deserialize objects from data that has been supplied by the user.  If you want to pass data between
two systems then you should rather use JSON or XML.

If you are going to unserialize then you should whitelist the classes that you want to allow to be instantiated.  You could change line 11 to the following, for example:

    $pickle = unserialize($data, ['allowed_classes' => [\UnserializeDemo\Pickle::class]]);
    
Lastly, you should always filter output.  You could prevent the XSS attack by using this line to output the string:

    echo "Hi, {$pickle->name}, the weather in London is " . filter_var($weather->weatherData, FILTER_SANITIZE_STRING) . PHP_EOL;
    
Note that this only mitigates XSS (which is the payload of this specific attack example).  It is not a mitigation against the attacker exploiting insecure deserialization.
