# The Beauty of PHP Value Objects

Imagine, if you will, a world where you're able to define a tailor-made type for domain objects that is always valid, type-safe, immutable, and easy to test. No more email addresses passed around as plain strings, nor associative arrays being passed around with potentially-undefined keys and unpredictable types.

In this session, we'll dive deep into PHP Value Objects: where are they useful, how do we write (and test!) them, and how do we ensure that the data they encapsulate is valid? Attendees will leave with a better understanding of domain modeling, Value Objects, and immutability.

Warning: once you start using proper value objects, you may never be able to go back to using anything else!

**[View slides](https://stevegrunwell.github.io/value-objects)**

## Resources

* [The Beauty of PHP Value Objects](https://stevegrunwell.com/blog/php-value-objects/) - Blog post that serves as the foundation for this talk
* [Value Objects in PHP 8](https://dev.to/cnastasi/value-objects-in-php-8-building-a-better-code-38k8) - A three-part series by Christian Nastasi that goes in-depth with value objects
* [Eloquent: Mutators and Casting](https://laravel.com/docs/master/eloquent-mutators) - Laravel documentation mutators and casting of properties
* [Value Object](https://martinfowler.com/bliki/ValueObject.html) — Article by Martin Fowler describing value objects
* [Writing Value Objects in PHP](https://dev.to/ianrodrigues/writing-value-objects-in-php-4acg) — Great article by Ian Rodrigues describing the inner-workings of value objects
* [Primitive Obsession](https://refactoring.guru/smells/primitive-obsession) - Refactoring-focused post discussing developers' desire to reduce everything to primitives

## Presentation History

* [PHP Tek 2025](https://phptek.io/) — May 20, 2025 ([joind.in](https://joind.in/talk/4f722))
* [Cascadia PHP 2024](https://cascadiaphp.com/) — October 25, 2024 ([PDF](https://github.com/stevegrunwell/value-objects/releases/download/cascadia-php-2024/slides.pdf), [joind.in](https://joind.in/talk/8c2f3))
