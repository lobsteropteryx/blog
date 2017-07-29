+++
date = "2017-07-16T00:00:00+00:00"
draft = true
title = "Test Time"
+++

I've spent a good deal of time helping teams adopt automated testing as a part of their workflow; one thing that I've heard over and over again from folks who aren't familiar with the methodology is something like the following:

> How much time do you estimate for writing unit tests?

Usually what people<sup>[1](#people)</sup> are looking for is something like, "I need X hours for the production code, and Y hours for writing tests for this feature."  If you're familiar with automated testing (and especially TDD), you probably recognize that this is the wrong way to look at it.  I've struggled with explaining this in the past, but I recently came a across a real example tha might be instructive.

We found a bug in some production code that was using a list component; when removing an item from the list, the `value` argument passed to the event handler was null, but we were looking for `value.id`.  In addition, the POST body for our API call expected the properties to be defined even if they weren't set, and to have a default value of `-1`.

The total time to discover, log, reproduce and find the root cause of this bug was on the order of a few **hours**.

The **production** code to fix the bug looked like this:

```javascript
static getCategoryValue(val) {
    let category = -1;
    if (val && val.id !== undefined && val.id !== null) {
      category = val.id;
    }
    return category;
}
```

The total time to write that function was on the order of a few **minutes**.

The **test** code to check the behavior looked like this:

```javascript
  describe('Category values', () => {
    it('Returns -1 if the value is null', () => {
      expect(PoiForm.getCategoryValue(null)).to.equal(-1);
    });

    it('Returns -1 if the value is undefined', () => {
      expect(PoiForm.getCategoryValue()).to.equal(-1);
    });

    it('Returns -1 if the id is null', () => {
      const val = {id: null};
      expect(PoiForm.getCategoryValue(val)).to.equal(-1);
    });

    it('Returns -1 if the id is undefined', () => {
      const val = {};
      expect(PoiForm.getCategoryValue(val)).to.equal(-1);
    });

    it('Returns the id if it is 0', () => {
      const val = {id: 0};
      expect(PoiForm.getCategoryValue(val)).to.equal(0);
    });

    it('Returns the id if it is 1', () => {
      const val = {id: 1};
      expect(PoiForm.getCategoryValue(val)).to.equal(1);
    });
  })
```

The total time to write the above tests was on the order of a few **seconds**.

Obviously these are order of magnitude estimates--your mileage may vary.  The point is, the test logic is very, very simple, and quick to write, while the time lost by an uncaught bug is very large--someone has to find and report the bug, create a ticket, etc--there's a lot of time and effort spent before we ever even touch the code.  And that's not even considering the time of our client or users--they could be losing entire days of productivity.

As a general rule, it doesn't make sense to compare tests and production methods by lines of code, nor to estimate them separately.  Ideally we'll be writing the tests and production code together, using TDD--that's the goal.  Adding a separate line item or estimate for unit testing is backwards, and moves us in the opposite direction.  Even worse, it can raise a red flag to the customer--a technically savvy client is going to think twice if they see "unit testing: 10 hours" on a contract.

If you're asked to estimate the time you spend writing tests, take a step back and try to reframe the conversation--when we're scoping a piece of work, we should really be talking in terms of features and defects, rather than specific technical tasks--business language, not technical jargon.  

<a id="people">1</a>: Project managers are people too!
