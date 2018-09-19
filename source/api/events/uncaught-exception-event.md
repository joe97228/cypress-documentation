---
title: uncaught:exception
---

The `uncaught:exception` event fires when an uncaught exception occurs in your application. Cypress will fail the test when this fires. Return `false` from this event and Cypress will not fail the test. Also useful for debugging purposes because the actual `error` instance is provided to you.

# Environment

Event | {% url "Browser" catalog-of-events#Browser-Events %} | {% url "Background Process" background-process %}
--- | --- | ---
`uncaught:exception` | {% fa fa-check-circle green %} |

# Arguments

**{% fa fa-angle-right %} error** ***(Object)***

The error instance.

**{% fa fa-angle-right %} runnable** ***(Object)***

The Mocha runnable this failed on.

# Usage

## In the browser

In a spec file or support file you can tap into the `uncaught:exception` event.

```javascript
Cypress.on('uncaught:exception', (err, runnable) => {

})
```

# Examples

## To turn off all uncaught exception handling

```javascript
// likely want to do this in a support file
// so it's applied to all spec files
// cypress/support/index.js

Cypress.on('uncaught:exception', (err, runnable) => {
  // returning false here prevents Cypress from
  // failing the test
  return false
})
```

## To catch a single uncaught exception

```javascript
it('is doing something very important', function (done) {
  // this event will automatically be unbound when this
  // test ends because it's attached to 'cy'
  cy.on('uncaught:exception', (err, runnable) => {
    expect(err.message).to.include('something about the error')

    // using mocha's async done callback to finish
    // this test so we prove that an uncaught exception
    // was thrown
    done()

    // return false to prevent the error from
    // failing this test
    return false
  })

  // assume this causes an error
  cy.get('button').click()
})
```

# See also

- {% url "Debugging" debugging %}
- {% url `fail` fail-event %}