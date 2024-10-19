<!-- .slide: class="title-slide" data-hide-footer -->
# The Beauty of PHP Value Objects

Steve Grunwell <!-- .element: class="byline" -->
[@stevegrunwell@phpc.social](https://phpc.social/@stevegrunwell)
[stevegrunwell.com/slides/value-objects](https://stevegrunwell.com/slides/value-objects)

---

## What is a Value Object?

An <u>immutable object</u> that<br>is <u>always in a valid state</u> and<br><u>encapsulates domain data</u>.

Note:

(In the broadest terms)

If you take nothing else away from this talk, remember:

1. Value objects are immutable
2. Value objects are always in a valid state

----

### A simple example: Age

```php
(new Age(37))->getValue();
#=> int(37)
```

```php
new Age(-1);
#=> Uncaught DomainException
```
<!-- .element: class="fragment" -->

Note:

For a super-simple example, let's imagine we have a want to represent an age; maybe we're not concerned with upper-bounds, but we know that someone's age must be greater than or equal to zero.

If we construct an Age object with an age like 37, we can retrieve the value and all is well. If we try to create an Age with value -1, however, the value object throws an exception because value objects are always in a valid state.

----

### Think custom types, not models

* Primitive types: int, float, bool, string,<br>null, array, object, resource
* <!-- .element: class="fragment" --> Value objects serve a similar purpose
    ```php
    new UnsignedInt(123);
    new NonEmptyString('hello, there!');
    ```

Note:

PHP has a number of primitive types, which are built into the language.

Think of value objects as creating your own primitive types for your domain: if we wanted a type to represent unsigned integers, for example, we might define an UnsignedInt value object class, which would throw an exception if given anything but a positive integer value. Or a `NonEmptyString()` value object class, which requires the value not only be a string but non-empty, too.

Remember that a value object models data, but is **not** a model/entity! Instead, think of value objects as custom types that you can add to represent the values of your models' properties

----

### Value Objects Must Be Immutable

What is the value of `$var`?

```php
$var = 4;
$var += 6;
```

<p class="fragment">What is the value of <code>int(4)</code>?</p>

Note:

As you may recall, one of the major rules is that a value object must be immutable.

Let's try a little thought exercise: var equals 4, then we add 6 to var. What is the new value of var?

This isn't a trick question, the value of $var is 10, right? But that's the value of the variable; did the integer 4 change?

No: 10 and 4 are two different values, in the same way that "example.com" and "example.org" are two different URLs. The values assigned to a model's _properties_ might change, but the values themselves do not change.

It's a bit "Ship of Theseus", but it demonstrates that a value object represents a *value*, not a fungible object.

By making our value objects immutable, we can also be sure that methods don't accidentally change values.

----

### Value Objects Must Be Valid

!["You Wouldn't Download a Car" meme, but instead it reads "You wouldn't increment a true"](resources/increment-a-true.jpg)<!-- .element: style="border: 75px solid #000;" -->

Note:

A value object is only useful if you can rely on it having valid domain data.

If you had a boolean value of 7 or an integer with value "taco", would that make sense?

In the same way, our value objects must always be in a valid state, which means our application must not allow invalid value objects to be constructed. Instead, we'll throw exceptions when given invalid data (more in a minute).

----

### Value Objects in the Wild

* DateTimeImmutable (but not DateTime)
<!-- .element: class="fragment" -->
* DateTimeZone, DateTimePeriod
<!-- .element: class="fragment" -->
* SensitiveParameterValue
<!-- .element: class="fragment" -->

Note:

One value object class you might be very familiar with is `DateTimeImmutable`, which is pre-defined by PHP and represents a date and time.

Note that the regular `DateTime` class cannot be considered a value object, because it's not immutable!

Similarly, PHP has the DateTimeZone and DateTimePeriod classes, which represent a time zone and period of time, respectively.

SensitiveParameterValue (added in PHP 8.2) - value object that wraps potentially sensitive data to keep it out of logs, debug statements, etc.

----

### Value Objects & Enums

* <!-- .element: class="fragment" --> Both apply domain constraints to data
* <!-- .element: class="fragment" --> Enums have a finite, known list of valid values
* <!-- .element: class="fragment" --> Value objects are dynamic, but must be valid

Note:

Both value objects and enums serve a similar purpose: applying domain constraints to data.

By definition, an enum has a finite, known list of possible values: countries, currency codes, user statuses, etc.

Value objects, on the other hand, must adhere to domain constraints but could have _any_ number of values. For instance, the value of a Username object might be any any string that consists of at least 2 but no more than 30 letters, numbers, or underscores.

If an enum makes more sense for your domain, by all means use that instead of a value object—both are useful ways to represent domain data.

----

### Value & Data Transfer Objects

* **DTOs:**
    * entity/model (or other structured data) <!-- .element: class="fragment" data-fragment-index="0" -->
    * no guarantee of validity <!-- .element: class="fragment" data-fragment-index="1" -->
* **Value Objects:**
    * single (possibly-complex) value <!-- .element: class="fragment" data-fragment-index="0" -->
    * must be valid <!-- .element: class="fragment" data-fragment-index="1" -->

Note:

While they're related, please don't confused Value Objects with Data Transfer Objects (DTOs).

DTOs are meant to provide a simplified representation of an entity or model; think how you might format the model to send it to the front end of your app.

Value objects, on the other hand, represent a single value.

Additionally, DTOs enforce the shape of the data, but **do not** necessarily guarantee that data aheres to domain requirements (e.g. you might have an age less than zero)

However, there's nothing stopping you from including value objects in your DTOs as long as you ensure that they can be serialized properly (more on this in a moment).


---

## Building Value Objects

Note:

Now that you understand that value objects are meant to represent actual values and must be both immutable and valid, let's discuss how we might build them in our own applications. But first, the elePHPant in the room...

----

### The Framework Dilema

_Should I use a framework for value objects?_

Note:

This is subjective, but I would say no: value objects should be as close to pure PHP as possible, and should be specialized to your application's domain.

You might find it helpful to define interfaces for consistency or traits for common properties, but I'd avoid frameworks and inheritance from any general-purpose value object class; if you need a complex inheritance scheme, you're probably overthinking things.

As we look at writing value objects, you'll see how simple they can be.

----

### A simple value object

```php [|7-11|3,13]
class Url
{
    private string $url;

    public function __construct(string $url)
    {
        if (!filter_var($url, FILTER_VALIDATE_URL)) {
            throw new \DomainException(
                sprintf('"%s" is not a valid URL!', $url)
            );
        }

        $this->url = $url;
    }
}
```
<!-- .element: class="hide-line-numbers growth-spurt" -->

Note:

Here's one of the simplest value objects we can build: a URL

The constructor uses PHP's built-in `filter_var()` function with the `FILTER_VALIDATE_URL` filter;
if it doesn't validate against RFC 2396, we'll throw a `DomainException`.

If it validates, we save the value to `$this->url`.

At this point, we cannot construct a `Url` object with an invalid URL, so the `$url` property will always be a valid URL.

----

### Retrieving values

```php
public function getValue(): string
{
    return $this->url;
}
```

Note:

In order to access our value, we might add a method like `getValue()`. In it, we can simply return `$this->url` since we know that it's valid, and the value will be a string because that's the underlying primitive type.

----

### Pro-tip: <span class="no-text-transform">__toString()</span>

```php
public function __toString(): string
{
    // This could also just be `return $this->url`.
    return $this->getValue();
}
```

```php
echo new Url('https://example.com');
#=> https://example.com
```
<!-- .element: class="fragment" -->

Note:

To make value objects even easier to work with, consider defining a __toString() method.

Any time someone tries to treat the value as a string, you can control what they'll see.

----

### Modern PHP, Better Value Objects

```php
class NonEmptyString
{
    public readonly string $value;

    // ...
}
```

```php
(new NonEmptyString('Put a bird on it!'))->value;
#=> string(17) "Put a bird on it!"
```
<!-- .element: class="fragment" -->

Note:

If you're using modern versions of PHP, you can take advantage of some nice features:

Readonly properties (PHP 8.1+) for direct property access (instead of a `getValue()`) method while staying immutable

This could be emulated with magic __get() methods, but setters should not be defined

Similarly, property hooks (coming in PHP 8.4) can simplify the retrieval of values, but should not be used for setters; all values should be passed via the constructor, then the object should be immutable!

----

### Constructor Property Promotion

```php [|3|5-9]
class Age
{
    public function __construct(public readonly int $value)
    {
        if ($value < 0) {
            throw new \RangeException(
                'Ages cannot be less than 0'
            );
        }
    }
}
```
<!-- .element: class="hide-line-numbers" -->

Note:

With constructor property promotion (was introduced in PHP 8.0), we can also do something like this to create a public, readonly property named value.

Note that the constructor method still runs, so we can throw an exception if the value is an integer less than zero.

----

### Complex Value Objects

```php [5-8|9-13]
use App\Enums\Currency;

class Price
{
    public function __construct(
        public readonly int $amount,
        public readonly Currency $currency
    ) {
        if ($amount < 0) {
            throw new \DomainException(
                'Cannot have a negative price.'
            );
        }
    }
}
```
<!-- .element: class="hide-line-numbers growth-spurt" -->

Note:

Be aware that value objects don't have to be limited to a single field; in fact, they become even more valuable when dealing with things like money.

In this case, we have a Price value object, which stores the amount (stored in the smallest denomination of the currency) and are using a Currency enum.

We get some validation for free by requiring the second argument be a valid Currency enum value, but we'll also make sure that a price is not less than 0.

Of course, if negative prices make sense for your domain, this validation rule might not apply. This is a good reminder that value objects are meant to be tailored to your domain.

----

### Compound Value Objects

```php
class Address
{
    public function __construct(
        public readonly StreetAddress $street_address,
        public readonly ?StreetAddress $extended_address,
        public readonly Locality $locality,
        public readonly Region $region,
        public readonly PostalCode $postal_code,
        public readonly Country $country
    ) {
        // ...
    }
}
```

Note:

Value objects may also be made up of other value objects, which can be really useful for things like addresses.

Here, we might have value objects defined for street addresses, localities/cities, regions/states, postal codes, and countries. We can then construct a full address that will always be valid.

Within our constructor, we also don't have to worry about validating any individual components, since we already know they're valid.

----

### Testing

```php [|3-10|11-15]
final class UrlTest extends TestCase
{
    public function testConstructorValidatesUrls(): void
    {
        $this->assertSame(
            'https://example.com',
            (new Url('https://example.com'))->getValue()
        );
    }

    public function testConstructorThrowsIfInvalid(): void
    {
        $this->expectException(\DomainException::class);
        new Url('💩');
    }
}
```
<!-- .element: class="hide-line-numbers growth-spurt" -->

Note:

One of the really great things about value objects is that it makes unit testing super easy: we can test the value object, and then know that everywhere it's used will be valid!

On one slide, we're able to fit the most bare-bones test case (would use a data provider, personally).

First, we validate that a URL with value "https://example.com" returns that same value when we call `getValue()`

Then, we test that the constructor throws if given an invalid URL.

----

### Let's see another example!

```php [|6-8|4]
class Email
{
    public function __construct(
        public readonly string $email
    ) {
        if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
            throw new \DomainException('Invalid email!');
        }
    }
}
```
<!-- .element: class="hide-line-numbers" -->

Note:

It's very common to be handling email addresses in our applications, but they're usually passed around as strings.

This is one of my favorite use-cases for value objects: defining a simple Email type like this ensures that we're always working with a valid email address.

In this case, we accept an email address as a string, then use the `FILTER_VALIDATE_EMAIL` filter to ensure that the email address adheres to the email address specifications. If not, we throw a `DomainException`.

Notice that we were able to use the public, readonly $email property instead of a `getValue()` method.

----

### Testing Email

```php
final class EmailTest extends TestCase
{
    public function testWithValidEmail(): void
    {
        $this->assertSame(
            'hello@example.com',
            new Email('hello@example.com')->email
        );
    }

    public function testWithInvalidEmail(): void
    {
        $this->expectException(\DomainException::class);
        new Email('example.com');
    }
}
```
<!-- .element: class="growth-spurt" -->

Note:

Again, our value object is pretty trivial to test. We'd probably want a few more test cases here, but we're able to verify that a valid email passes, and an invalid email fails.

---

## Using value objects

Note:

We've discussed how to build value objects, but let's see how they make things easier throughout our applications.

----

### Typehinting

```php
public function sendMessage(
    Email $email,
    string $subject,
    string $body
) {
    // ...
}
```

```php
$this->sendMessage('example.com', /* ... */);
#=> Fatal error: Uncaught TypeError: SomeClass::sendMessage(): Argument #1 ($email) must be of type Email, string given...
```
<!-- .element: class="fragment" -->

Note:

Perhaps the biggest benefit of value objects is that suddenly we can typehint them.

Imagine we have a `sendMessage()` method that sends an email; it has three arguments:

email, which must be an Email value object (and thus a valid email address), then subject and body, which both must be strings (though we could create value objects for these, too).

If anyone tries to pass a plain string instead of an Email value, they'll get a `TypeError`. If they try to construct the value with an invalid email address, they'll get a `DomainException`.

As long as we have an Email object, we know that it's valid!

----

### Property Typehints

```php
class User
{
    public Email $email;

    // ...
}
```

Note:

We're not only type-hinting method arguments, either. Thanks to typed properties, we can ensure that this user class' email property is always an instance of our Email value object class.

Then, because we know our Email value objects are always valid, we know that a user's email is valid, too!

Speaking of valid...

----

### Validation

```php
try {
    $username = new Username($request->input('username'));
} catch (\DomainException $e) {
    // This username doesn't fit our requirements
}

// Other validation
```

Note:

Once we've defined value objects, we can use them for tasks like validation: the value object should already know the rules (min/max length, characters, etc.), so we could take user input and see if it passes.

The benefit here is that your username validation logic is in one, easily-tested location. If the rules change, there's one place that needs to be updated.

----

### Validation Helper

```php
class Username
{
    // ...

    public static function isValid(string $value): bool
    {
        try {
            new self($value);
        } catch (\DomainException $e) {
            return false;
        }

        return true;
    }
}
```
<!-- .element: class="growth-spurt" -->

Note:

If you find yourself doing a lot of try/catch validation blocks, you might also consider something like a static `isValid()` helper method on your value object class; here, it attempts to construct an instance of itself with a given value and, if it catches a DomainException, return false.

Otherwise, if it constructed without issue, it returns true.

Note that this is a static method, because we can't have an invalid instance of our value object classes.

----

### Comparing Value Objects

```php [1-2|4-5|7-8]
$age1 = new Age(40);
$age2 = new Age(40);

$age1 == $age2;
#=> bool(true);

$age1 === $age2;
#=> bool(false)
```
<!-- .element: class="hide-line-numbers" -->

Note:

One drawback to value objects is that we need to be clever when comparing them, because each time we call the constructor we get a new instance of the object.

Given two instances of Age with the same value of 40, they will evaluate as loosely equal but, because they are separate instances, will fail a strict equality check.

There are a few ways we can address this:

----

#### Comparator Methods

```php
public function isEqualTo(Age $age): bool
{
    return $age->value === $this->value;
}
```

```php
$age1->isEqualTo($age2);
#=> bool(true)
```
<!-- .element: class="fragment" -->

Note:

The first approach is to create methods like `isEqualTo()`, `isGreaterThan()`, etc. on the value object class. We typehint the argument as another instance of the value object, then compare the underlying values.

The methods you might define here will vary based on the types of comparisons that are necessary for your domain.

----

#### More Comparator Methods

```php
public function isSameBuilding(Address $address): bool
{
    return $this->street_address == $address->street_address
        && $this->region == $address->region
        && $this->locality == $address->locality
        && $this->postal_code == $address->postal_code;
}
```

Note:

If you're dealing with complex or compound value objects, you might also consider comparators that operate on multiple values.

In this instance, we might have an `isSameBuilding()` method that checks the key values but ignores things like apartment/suite number.

Maybe you're dealing with money and want to check that two amounts are in the same currency.

These are powerful methods that clean up your domain logic in a way that's both clear and easy to test.

----

#### Value comparisons

```php [1-2|4-5|7-8]
$age1->value === $age2->value;
#=> bool(true)

$age1->checksum() === $age2->checksum();
#=> bool(true)

(string) $age1 === (string) $age2;
#=> bool(true)
```
<!-- .element: class="hide-line-numbers" -->

Note:

Another approach would be comparing the underlying values directly.

If you have multiple properties, you might consider something like a `checksum()` method, which creates a determinate value.

If you implemented a `__toString()` method, you could also compare them as strings.

----

### Manipulation

```php [1-2|4-12]
$age1 = new Age(37);
$age2 = $age1->increment();

var_dump($age1, $age2)
#=> object(Age)#1 (1) {
#     ["value":"Age":private]=>
#     int(37)
#   }
#   object(Age)#2 (1) {
#     ["value":"Age":private]=>
#     int(38)
#   }
```
<!-- .element: class="hide-line-numbers" -->

Note:

Remember that value objects need to be immutable, but that doesn't mean you can't manipulate values.

In this case, we've defined an `Age` object with a value of 37. Now imagine we have an increment method: it **must** return a _new_ `Age` instance, because value objects are immutable.

This is the same thing that happens when you try to manipulate a `DateTimeImmutable` object: the methods defined by `DateTimeInterface` are implemented, but the object returned is not the same instance on which you called them, since that object is immutable.

----

### Serialization

```php
class Email implements \JsonSerializable
{
    // ...

    public function jsonSerialize(): string
    {
        return $this->value;
    }
}
```

```php
json_encode(new Email('test@example.com'));
#=> "test@example.com"
```
<!-- .element: class="fragment" -->

Note:

There are two major ways that PHP can serialize objects:

1. PHP serialization, which stores the properties of the object in an encoded string
2. JSON serialization, which tends to be faster and more interoperable with other systems, created via `json_encode()`

By default, PHP serialization tends to work well for value objects, since it's explicitly enumerating the object properties.

It can be helpful to implement the `JsonSerializable` interface in your value objects, which let you determine what happens when the value object is JSON-encoded (default is an empty object).

For simple value objects, this may just be the underlying, primitive value.

----

### Serialization

```php
class Coordinates implements \JsonSerializable
{
    // ...

    public function jsonSerialize(): array
    {
        return [
            'lat' => $this->latitude,
            'lng' => $this->longitude,
        ];
    }
}
```

```php
json_encode(new Coordinates(45.507, -122.683));
#=> {"lat":45.507,"lng":-122.683}
```
<!-- .element: class="fragment" -->

Note:

If you have a more complex value object, like coordinates, you might return an array representing relevant values.

----

### Hydrating from database

```php [1|2-4,9|5-8]
$query = 'SELECT DISTINCT locality, region FROM addresses';
$dbh->query($query)
    ->fetchAll(
        PDO::FETCH_FUNC,
        fn ($locality, $region) => new CityState(
            new City($locality),
            new State($region)
        )
    );
```
<!-- .element: class="hide-line-numbers" -->

Note:

If you're interacting with a database, automatically retrieving specific columns as value objects can be immensely helpful.

If you're using PDO directly, you might define a fetch function like this that automatically casts results to value objects.

In this case, we're constructing a CityState object, which is a compound value object that consists of City and State objects. The query will result in one CityState instance for each unique combination of locality and region (city and state) in our database.

----

### Eloquent Casting

```sh
php artisan make:cast HexColor
```

```php
use App\Casts\HexColor;
use Illuminate\Database\Eloquent\Model;

class Profile extends Model
{
    protected function casts(): array
    {
        return [
            'background_color' => HexColor::class,
        ];
    }
}
```
<!-- .element: class="fragment" -->

Note:

If you're using Laravel, Eloquent has a built-in way to cast model properties.

If you've noticed that columns like created_at and updated_at come back as Carbon objects, this should be very familiar (same approach).

First, we need to generate a new implementation of the `Illuminate\Contracts\Database\Eloquent\CastsAttributes` interface, which we can do with this Artisan command.

It will generate a HexColor class that implements CastsAttributes, which has two methods:

1. `get()` will return a value object
2. `set()` will extract the underlying value so that it may be stored in the database

Then, in our model, add the property to array returned from our `casts()` method, ensuring it references the CastsAttributes implementation. Now, any time we reference the background_color attribute on our model, we'll get a HexColor value object.

If you're using Doctrine, look at "Custom Mapping Types" and "Embedables"

---

## Adding Value to your<br>Value Objects

Note:

I hope by now you have a solid grasp on value objects and how they're useful.

Let's wrap up by looking at ways to get even more out of your value objects.

----


### Specialized Getters

```php
class Url
{
    // ...

    public function getProtocol(): string
    {
        // "http", "https", etc.
    }

    public function getHost(): string
    {
        // "example.com", "example.org", etc.
    }
}
```

Note:

Once we've encapsulated the URL in a value object, we can also add methods to retrieve things like the protocol, host, path, and everything else that makes up a URL.

This means you won't have to litter your codebase with `parse_url()`, and it allows you to include logic like null coalescing, so `getQuery()` could return an empty string if no such value is set.

----

### Pro-tip: Don't Repeat Yourself!

```php [3,10-13|5-8]
class Url
{
    private array $parsed;

    public function getQuery(): string
    {
        return $this->parse()['query'] ?? '';
    }

    private function function parse(): array
    {
        return $this->parsed ??= parse_url($this->url);
    }
}
```
<!-- .element: class="hide-line-numbers" -->

Note:

When implementing these methods, we might call a `parse()` method that parses + caches parsed url, returning an array of its components.

This ensures that we're only calling `parse_url()` once, not every time we try to retrieve an element. Better yet, we don't have to actually call `parse_url()` at all until the first time we need to retrieve a component.

Please remember that just-in-time parsing methods like this should not be used for the actual validation of the data, as that should happen at the time the value object is instantiated!

----

### Testing Getters

```php
public function testGetHost(): void
{
    $this->assertSame(
        'example.com',
        (new Url('https://example.com'))->getHost()
    );
}
```

Note:

If we've added custom getters, we can easily test these, too!

----

### Factory Methods

```php [|8-12]
class Age
{
    // ...

    public static function fromBirthdate(
        DateTimeInterface $bday
    ): self {
        $age = new DateTimeImmutable('now')
            ->diff($bday)
            ->format('%Y');

        return new self((int) $age);
    }
}
```
<!-- .element: class="hide-line-numbers" -->

Note:

Adding factory methods to your value objects can make it easier to construct them.

For instance, we could calculate someone's age by taking their birthday (as a datetime object) and comparing it to now. In this case, calculate the difference in years, then cast that as an integer and construct a new Age value object.

----

### Objects, Not Arrays

```php
$request_body = [
    'name' => 'Taco',
    'type' => 'cat',
    'age'  => 2,
];
```

```php
new PetRequest(...$request_body);
```
<!-- .element: class="fragment" -->

```php
new PetRequest(
    name: $request_body['name'],
    type: $request_body['type'],
    age: $request_body['age']
);
```
<!-- .element: class="fragment" -->

Note:

This doesn't strictly apply to value objects, but is worth mentioning:

Associative arrays have been abused for a long time in PHP; whether it's the body of a POST request or some sort of pseudo-struct, associative arrays offer no type-safety, no immutability, and are a common source of undefined array key warnings.

Imagine we have this $request_body, which has a few details about my cat in an associative array.

As of PHP 8.1, associative arrays may be unpacked into named arguments, so in one line we could turn this array into a `PetRequest` value object. Note that this would work in versions before 8.1 too, but the argument order would matter.

Now we have a valid, immutable object representing the request body!

---

## Where Else Might You Use Value Objects?

![David S. Pumpkins (Tom Hanks) and his two skeleton B-Boys from Saturday Night Live, with the caption "Any questions?"](resources/david-s-pumpkins.jpg)
<!-- .element: class="fragment" -->

Note:

I've shown you a range of different applications for value objects, but I'm sure new ones have popped into your heads.

Where else might you find value objects useful?

As you've seen, value objects can be a beautiful way to encapsulate your domain logic into easily-maintained, easily-tested chunks.

REMEMBER TO REPEAT THE QUESTION!!

---

<!-- .slide: data-hide-footer -->

## Thank You!

Steve Grunwell<br>
<span style="font-size: .75em;">Staff Software Engineer, Mailchimp</span>

[stevegrunwell.com/slides/value-objects](https://stevegrunwell.com/slides/value-objects)<!-- .element: class="slides-link" -->
