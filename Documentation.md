Almost every programming language has a date/time formatting function, which works something like this:

```
date_format(timestamp, "%d/%m/%Y");
```

You can use this function to also format intervals, but it's not really meant for formatting intervals.  A great degree of functionality doesn't make sense in the context of intervals, like month and weekday names and 12-hour time.  And a great deal of functionality is missing, like expressing the interval in the number of days.

A special interval formatting function is therefore needed, and this is the gap that jintervals is trying to fill.

### Syntax ###

jintervals format string consists of plain text and codes between `{` and `}`.  The plain text is left untouched, but codes are replaced by interval values.

### Units ###

Each code begins with letter, that determines the interval unit.  Currently four units are supported:

  * `{D}` - days
  * `{H}` - hours
  * `{M}` - minutes
  * `{S}` - seconds

For example we can format 65 seconds either as minutes or as seconds:

```
jintervals(65, "{M} min, {S} sec"); --> 1 min, 65 sec
```

This is fine, when we just want to format our interval in either seconds or minutes, but it's not very useful when we want to format it in both seconds and minutes.  This is where the case comes into play.

### Case ###

When you start a scentence, you should start with an uppercase letter.  The same holds true in jintervals.  The first interval code should be in uppercase and all the following in lowercase.  Let's modify the previous example to conform to this rule:

```
jintervals(65, "{M} min, {s} sec"); --> 1 min, 5 sec
```

Exactly what was needed.  This is because `{s}` only returns a number between 0 and 59.  Likewise act other lowercased units:

  * `{d}` - days 0..
  * `{h}` - hours 0..23
  * `{m}` - minutes 0..59
  * `{s}` - seconds 0..59

`{d}` acts currently just like `{D}`, having no upper bound.  But you should nevertheless use `{D}` at the beginning of format string, to avoid problems when we at one day decide to introduce an upper bound for it (e.g. 365 days).

### Decimal places ###

Often precise number of decimal places is needed.  For example, normally we don't want this:

```
jintervals(65, "{M}:{s}"); --> 1:5
```

Instead we would like it to display `01:05`.  To specify the minimum number of decimal places, you repeat the unit symbol:

```
jintervals(65, "{MM}:{ss}"); --> 01:05
jintervals(65, "{MMM}:{sss}"); --> 001:005
```

### Unit abbrevations ###

Sometimes you would like to put a single letter behind value to designate the unit.  This could be done like this:

```
jintervals(65, "{M}m, {s}s"); --> 1m, 5s
```

But jintervals has a nice shortcut built in just for that:

```
jintervals(65, "{M.}, {s.}"); --> 1m, 5s
```

This example also has the advantage of being easily localized (see below).

Why the dot?  Because abbrevations often end with dot, this should make it easier to remember.

### Unit names ###

Sometimes you would like to spell it out large.  Say, "1 minute and 5 seconds".  Taking also into account when you need to use plural and singular form.  This can easily get pretty messy.  Luckily jintervals has this feature built in:

```
jintervals(65, "{Minutes} and {seconds}"); --> 1 minute and 5 seconds
```

Yes, you just write the whole name of unit to, er... get the whole name of unit.  Here's a whole list of those full-unit-name codes:

  * `{days}`
  * `{hours}`
  * `{minutes}`
  * `{seconds}`

### Optional units ###

Our previous example has one noticable problem.  What happens when the number of minutes is zero?

```
jintervals(5, "{Minutes} and {seconds}"); --> 0 minutes and 5 seconds
```

The information about zero minutes is quite unneccessary.  Wouldn't it be so much better if it just displayed "5 minutes".  Luckily again, jintervals can handle it.

```
jintervals(5, "{Minutes?} {seconds}"); -->  5 seconds
```

Placing a question mark after unit name will make it disappear when it's zero.

But this was cheating.  We removed the " and " between the two completely.  Can you do it so, that " and " following minutes is shown and hidden together with them?

Oh yes.  You can place any text you want to act that way behind the question mark:

```
jintervals( 5, "{Minutes? and }{seconds}"); --> 5 seconds
jintervals(65, "{Minutes? and }{seconds}"); --> 1 minute and 5 seconds
```

### {Greatest} ###

Sometimes we only care about the highest unit.  For example when user posted a message 2 hours, 23 minutes and 15 seconds ago, we really only care about hours.  Again, jintervals tries to help you with this problem.

In addition to units mentioned before, there is a special unit `{Greatest}`, which will always select the greatest unit with non-zero value.  For example:

```
jintervals( 5, "{Greatest} ago"); --> 5 seconds ago
jintervals(65, "{Greatest} ago"); --> 1 minute ago
```

`{G}` acts like any other unit (except that there is no difference between `{g}` and `{G}`):

```
jintervals( 5, "{G.}"); --> 5s
jintervals(65, "{GGGreatest}"); --> 001 minute
```

### A word about rounding ###

When you display a delay given in seconds in minutes or hours, there is a loss of precision. Jintervals tries to provide the most accurate number by rounding the smallest unit.  For example:

```
jintervals(10, "{Minutes}"); --> 0 minutes
jintervals(29, "{Minutes}"); --> 0 minutes
jintervals(30, "{Minutes}"); --> 1 minute
jintervals(59, "{Minutes}"); --> 1 minute
jintervals(60, "{Minutes}"); --> 1 minute
jintervals(90, "{Minutes}"); --> 2 minutes
```

The rouding even works as you would expect with {Greatest}:

```
jintervals(30, "{Greatest}"); --> 30 seconds
jintervals(59, "{Greatest}"); --> 59 seconds
jintervals(60, "{Greatest}"); --> 1 minute
jintervals(90, "{Greatest}"); --> 2 minutes
jintervals(60*30, "{Greatest}"); --> 30 minutes
jintervals(60*59, "{Greatest}"); --> 59 minutes
jintervals(60*60, "{Greatest}"); --> 1 hour
```

### Escaping ###

To use `{` inside plain text you will need to escape it with backslash.  And to use backslash, you need to escape it with a basckslash too.  But because JavaScript uses backslashes for escaping too, you will need to double-escape your backslashes:

```
// what you will write:
jintervals( 5, "\\{ {seconds} \\\\");

// what jintervals will see:
jintervals( 5, "\{ {seconds} \\");

// what the output will be:
{ 5 seconds \
```

### Localization ###

To get and set the current locale, use the jintevals.locale() function:

```
// start with default locale
jintervals.locale();         // --> "en_US"
jintervals(10, "{seconds}"); // --> "10 seconds"

// switch to Russian
jintervals.locale("ru_RU");  // --> "ru_RU"
jintervals(10, "{seconds}"); // --> "10 секунд"

// switch to Finnish
jintervals.locale("fi_FI");  // --> "fi_FI"
jintervals(10, "{seconds}"); // --> "10 sekunttia"

// switch to Estonian
jintervals.locale("et_EE");  // --> "et_EE"
jintervals(10, "{seconds}"); // --> "10 sekundit"

// switch to Lihtuanian
jintervals.locale("lt_LT");  // --> "lt_LT"
jintervals(10, "{seconds}"); // --> "10 sekundžų"

// switch back to default
jintervals.locale("en_US");  // --> "en_US"
```

Currently these are the only supported locales.  But it's easy to add new ones by your own.  See the source code.