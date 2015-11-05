==================
The Currency Field
==================

The currency field is a numerical field, specifically designed to manage
monetary values.

  >>> from z3c.currency import field, interfaces

  >>> price = field.Currency(
  ...    title=u'Price',
  ...    description=u'The price of the item.')

Besides the common attributes, the currency field also provides two additional
attributes, the precision and unit. The precision is intended to allow for
specifying whether the value is provided whole units or 1/100th of the whole
unit -- in the US dollar and cents. By default this field is set to cents:

  >>> price.precision is interfaces.CENTS
  True

It can be set to be dollars:

  >>> price.precision = interfaces.DOLLARS
  >>> price.precision is interfaces.DOLLARS
  True

For financial applications, we also sometimes needsub-cents:

  >>> price.precision = interfaces.SUBCENTS
  >>> price.precision is interfaces.SUBCENTS
  True


Note: Is there a more "internationalized" word for the whole unit of a
currency?

The unit specifies the symbol of the currency used. It is also used for
formatting the numerical value to a string.

  >>> price.unit
  u'$'
  >>> price.unit = u'€'
  >>> price.unit
  u'\xe2\x82\xac'

Of course, both of those attributes are available as constructor arguments:

  >>> price = field.Currency(
  ...    title=u'Price',
  ...    description=u'The price of the item.',
  ...    precision=interfaces.DOLLARS,
  ...    unit=u'€')

  >>> price.precision is interfaces.DOLLARS
  True
  >>> price.unit
  u'\xe2\x82\xac'

Let's now have a look at the validation. First of all, the value must always
be a decimal:

  >>> import decimal
  >>> price.validate(decimal.Decimal('12'))

  >>> price.validate(12)
  Traceback (most recent call last):
  ...
  WrongCurrencyType: int

  >>> price.validate(12.0)
  Traceback (most recent call last):
  ...
  WrongCurrencyType: float

Also, when the precision is set to DOLLARS as it is the case here, the value
must be a whole number:

  >>> price.validate(decimal.Decimal('12'))

  >>> price.validate(decimal.Decimal('12.01'))
  Traceback (most recent call last):
  ...
  IncorrectValuePrecision: 0

  >>> price.validate(decimal.Decimal('12.00'))
  Traceback (most recent call last):
  ...
  IncorrectValuePrecision: 0

When the precision is set to cents,

  >>> price.precision = interfaces.CENTS

then values only with two decimal places are accepted:

  >>> price.validate(decimal.Decimal('12.00'))

  >>> price.validate(decimal.Decimal('12'))
  Traceback (most recent call last):
  ...
  IncorrectValuePrecision: 1

  >>> price.validate(decimal.Decimal('12.0'))
  Traceback (most recent call last):
  ...
  IncorrectValuePrecision: 1

If we allow sub-cents,

  >>> price.precision = interfaces.SUBCENTS

any precision is allowed:

  >>> price.validate(decimal.Decimal('12.0'))
  >>> price.validate(decimal.Decimal('12'))
  >>> price.validate(decimal.Decimal('12.00001'))

If the field is not required, ...

  >>> price.required = False

let's make sure that the validation still passes.

  >>> price.validate(None)

Note that the ``IFromUnicode`` interface is purposefully not supported:

  >>> price.fromUnicode
  Traceback (most recent call last):
  ...
  AttributeError: 'Currency' object has no attribute 'fromUnicode'


``z3c.form`` Support
--------------------

This package also provides support for integration with the ``z3c.form``
package. In particular it implements a data converter from the ``Currency``
field to any widget accepting a unicode string.

  >>> from z3c.currency import converter
  >>> conv = converter.CurrencyConverter(price, None)
  >>> conv
  <DataConverter from Currency to NoneType>

The converter easily produces a string from any value:

  >>> conv.toWidgetValue(decimal.Decimal(12))
  u'12'
  >>> conv.toWidgetValue(decimal.Decimal(1200))
  u'1,200'
  >>> conv.toWidgetValue(decimal.Decimal(-12))
  u'-12'
  >>> conv.toWidgetValue(decimal.Decimal('-12.0'))
  u'-12.00'
  >>> conv.toWidgetValue(decimal.Decimal('-12.00'))
  u'-12.00'

Note that always two decimal places are printed. You can also set the
precision to DOLLARS:

  >>> conv.field.precision = interfaces.DOLLARS

  >>> conv.toWidgetValue(decimal.Decimal(12))
  u'12'
  >>> conv.toWidgetValue(decimal.Decimal('12.00'))
  u'12'

Let's try sub-cents as well:

  >>> conv.field.precision = interfaces.SUBCENTS

  >>> conv.toWidgetValue(decimal.Decimal('12.00'))
  u'12.00'
  >>> conv.toWidgetValue(decimal.Decimal('12'))
  u'12'
  >>> conv.toWidgetValue(decimal.Decimal('12.0001'))
  u'12.0001'

If the value is missing, then handle it gracefully.

  >>> conv.toWidgetValue(None)
  u''

Let's now parse a value. The parser is a little bit flexible, not only
accepting the output values, ...

  >>> conv.field.precision = interfaces.CENTS
  >>> conv.toFieldValue(u'12')
  Decimal('12.00')
  >>> conv.toFieldValue(u'1,200')
  Decimal('1200.00')
  >>> conv.toFieldValue(u'-12')
  Decimal('-12.00')
  >>> conv.toFieldValue(u'-12.00')
  Decimal('-12.00')

  >>> conv.field.precision = interfaces.DOLLARS
  >>> conv.toFieldValue(u'12')
  Decimal('12')
  >>> conv.toFieldValue(u'12.00')
  Decimal('12')

  >>> conv.field.precision = interfaces.SUBCENTS
  >>> conv.toFieldValue(u'12')
  Decimal('12')
  >>> conv.toFieldValue(u'12.00')
  Decimal('12.00')
  >>> conv.toFieldValue(u'12.0000')
  Decimal('12.0000')
  >>> conv.toFieldValue(u'12.0001')
  Decimal('12.0001')

but also other input values:

  >>> conv.toFieldValue(u'1200')
  Decimal('1200')

If the browser sends an empty string, then handle it gracefully.

  >>> conv.toFieldValue(u'')

