This document contains miscellaneous best practice guidelines for writing code. Not everything is covered here, as linting handles lots of that for us.

You should attempt to match the guidelines in your code. You should review code with the guidelines in mind.

# General guidelines

## Lint on save

There is an eslint configuration within the repo. We recommend you set your editor to format on save, to ensure the lint configuration is applied before a pull request is opened.

## Functions, not classes

By default, the code we write uses the functional programming paradigm. On occasion classes will be used, but we mostly avoid them at all cost.

If you want to know more about this, you can read [@Antman261](https://github.com/Antman261)'s blog post here: [https://antman-does-software.com/functional-typescript-in-production-systems](https://antman-does-software.com/functional-typescript-in-production-systems).

It’s also worth reading [@Antman261](https://github.com/Antman261)'s blog post on when to use map, forEach, and reduce: [https://antman-does-software.com/are-you-using-map-foreach-and-reduce-wrong](https://antman-does-software.com/are-you-using-map-foreach-and-reduce-wrong).

When declaring multiple functions in a file, we prefer to order the main function at the top and helper functions below, that way you can read from the highest level of abstraction down.

## Unions, not enums

We don’t use enums at all, outside of generated code. Instead we use something like:

```typescript  
type fruit = 'APPLE' | 'BANANA' | 'ORANGE'  
```  
Or  
```typescript  
const COUNTRIES = {  
 AU: 'AUSTRALIA',  
 US: 'UNITED_STATES',  
} as const  
```

## Types, not interfaces

Interfaces are useful for classes, while types are suited to functional programming. Types allow us to get intersections, unions, and more. We prefer to declare types close to their implementation (same file if possible).

## Named default exports

We don’t have this as an ESLint rule (yet\!), but we want to give names to all our default exports.

```typescript  
export default function doTheThing //not export default function  
```

## Declare return types

We give warnings for this with ESLint but we don’t require it, however we highly recommend you do. Your functions should declare what they will return. This makes sure your function always returns what you expect, and you don’t accidentally return something unintended. It also makes the intention of your function clearer.

## Small functions

In general, we try to keep functions small. A few lines, opposed to many tens or hundreds of lines. Functions should have a clear purpose. If your functions start to get big, refactor, and make the intention of what’s happening in that section of code clearer.

## Avoid try/catch

We aim to avoid using try/catch statements. [@Antman261](https://github.com/Antman261)'s has a blog post on this here: [https://antman-does-software.com/stop-catching-errors-in-typescript-use-the-either-type-to-make-your-code-predictable](https://antman-does-software.com/stop-catching-errors-in-typescript-use-the-either-type-to-make-your-code-predictable).

But note that we use neverthrow for error handling. You can read more about that here: [https://github.com/supermacro/neverthrow](https://github.com/supermacro/neverthrow)

## Tests

Write them. Practice TDD. [@Antman261](https://github.com/Antman261)'s has a blog post on tests too, that’s worth reading: [https://antman-does-software.com/applying-googles-testing-methodology-to-functional-domain-driven-design-for-scalable-testing](https://antman-does-software.com/applying-googles-testing-methodology-to-functional-domain-driven-design-for-scalable-testing). Tests should ideally always use fixed values.

## Small PRs

Keep your PRs small, long lived code is a liability, and the larger the PR the longer lived the branch is.

## Comments

We try to avoid comments within the codebase. Your code should be self explanatory through naming and small functions that do one thing. Comments mostly become outdated and require upkeep. They don’t provide anything that good code doesn’t. 

We do use comments if there’s something that cannot be explained through naming and obvious functions, such as a ‘why’ based around a quirk of a system.

## Use guards and return early

When writing code, nesting lines inside if statements and other conditions can increase cognitive load and reduce readability. Rather than nesting large amounts of code in if statements, we prefer to use [guards](https://betterprogramming.pub/refactoring-guard-clauses-2ceeaa1a9da) to return early, which is usually the negative case.  
For example, don't do this:  
```typescript  
function acceptOrder(order, warehouse) {
  const result = deriveCanAcceptOrder(order, warehouse);
  if (result.outcome === 'SUCCESS') {
    const updatedOrder = orderRepo.update(order.id, { status: 'ACCEPTED'});
    const binsPrepared = await autostore.prepareBins(result.order.requiredBins);
    if (binsPrepared.outcome === 'SUCCESS') {
      await raiseEvent({ type: 'ORDER_ACCEPTED', orderId: result.order.id });
      return result;
    }
    await raiseEvent({ type: 'ORDER_REJECTED', orderId: result.order.id });
    updatedOrder.rollback();
  }
  await raiseEvent({ type: 'ORDER_REJECTED', orderId: result.order.id });
  return result
}
```
Instead, we look to do:   
```typescript
function acceptOrder(order, warehouse) {
  const result = deriveCanAcceptOrder(order, warehouse);
  if (result.outcome !== 'SUCCESS') {
    await raiseEvent({ type: 'ORDER_REJECTED', orderId: result.order.id });
    return result
  }
  const updatedOrder = orderRepo.update(order.id, { status: 'ACCEPTED'});
  const binsPrepared = await autostore.prepareBins(result.order.requiredBins);
  if (binsPrepared.outcome !== 'SUCCESS') {
    updatedOrder.rollback()
    await raiseEvent({ type: 'ORDER_REJECTED', orderId: result.order.id });
    return;
  }
  await raiseEvent({ type: 'ORDER_ACCEPTED', orderId: result.order.id });
  return result;
}
```
The code above is just a contrived example.

## Type and schema

When defining type and schema. Naming convention is

* Schema should be camel case. e.g `countryCode`  
* Type should be pascal case. e.g `CountryCode`

# Additional resources

[@Antman261](https://github.com/Antman261)'s blog has a bunch of useful thoughts and insights that often influence how we do things, and is in general just worth a read: [https://antman-does-software.com/](https://antman-does-software.com/)  
   
