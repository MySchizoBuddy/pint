.. _tutorial:


Tutorial
========

Pint has the concept of Unit Registry, an object within which units are defined and handled. You start by creating your registry::

   >>> from pint import UnitRegistry
   >>> ureg = UnitRegistry()

If no parameter is given to the constructor, the unit registry is populated with the default list of units and prefixes.
You can now simply use the registry in the following way::

   >>> distance = 24.0 * ureg.meter
   >>> print(distance)
   24.0 meter
   >>> time = 8.0 * ureg.second
   >>> print(time)
   8.0 second
   >>> print(repr(time))
   <Quantity(8.0, 'second')>

In this code `distance` and `time` are physical quantities objects (`Quantity`). Physical quantities can be queried for the magnitude and units::

   >>> print(distance.magnitude)
   24.0
   >>> print(distance.units)
   meter

and can handle mathematical operations between::

   >>> speed = distance / time
   >>> print(speed)
   3.0 meter / second

As unit registry knows about the relationship between different units, you can convert quantities to the unit of choice::

   >>> speed.to(ureg.inch / ureg.minute )
   <Quantity(7086.61417323, 'inch / minute')>

This method returns a new object leaving the original intact as can be seen by::

   >>> print(speed)
   3.0 meter / second

If you want to convert in-place (i.e. without creating another object), you can use the `ito` method::

   >>> speed.ito(ureg.inch / ureg.minute )
   <Quantity(7086.61417323, 'inch / minute')>
   >>> print(speed)
   7086.61417323 inch / minute

If you ask Pint to perform and invalid conversion::

   >>> speed.to(ureg.joule)
   Traceback (most recent call last):
   ...
   pint.pint.DimensionalityError: Cannot convert from 'inch / minute' (length / time) to 'joule' (length ** 2 * mass / time ** 2)


In some cases it is useful to define physical quantities objects using the class constructor::

   >>> Q_ = ureg.Quantity
   >>> Q_(1.78, ureg.meter) == 1.78 * ureg.meter
   True

(I tend to abbreviate Quantity as `Q_`) The in-built parse allows to recognize prefixed and pluralized units even though they are not in the definition list::

   >>> distance = 42 * ureg.kilometers
   >>> print(distance)
   42 kilometer
   >>> print(distance.to(ureg.meter))
   42000.0 meter

If you try to use a unit which is not in the registry::

   >>> speed = 23 * ureg.snail_speed
   Traceback (most recent call last):
   ...
   pint.pint.UndefinedUnitError: 'snail_speed' is not defined in the unit registry

You can add your own units to the registry or build your own list. More info on that :ref:`defining`


String parsing
--------------

Pint can also handle units provided as strings::

   >>> 2.54 * ureg['centimeter']
   <Quantity(2.54, 'centimeter')>

or via de `Quantity` constructor::

   >>> Q_(2.54, 'centimeter')
   <Quantity(2.54, 'centimeter')>

Numbers are also parsed::

   >>> Q_('2.54 * centimeter')
   <Quantity(2.54, 'centimeter')>

This enables you to build a simple unit converter in 3 lines::

   >>> input = '2.54 * centimeter to inch' # this is obtained from user input
   >>> src, dst = input.split(' to ')
   >>> Q_(src).to(dst)
   <Quantity(1.0, 'inch')>

Take a look at `qconvert.py` within the examples folder for a full script.


String formatting
-----------------

Pint's physical quantities can be easily printed::

   >>> accel = 1.3 * ureg['meter/second**2']
   >>> 'The str is {:!s}'.format(accel) # The standard string formatting code
   'The str is 1.3 meter / second ** 2'
   >>> 'The repr is {:!r}'.format(accel) # The standard representation formatting code
   'The repr is <Quantity(1.3, 'meter/second**2')>'
   >>> 'The magnitude is {0.magnitude} with units {0.units}'.format(accel) # Accessing useful attributes
   'The magnitude is 1.3 with units meter / second ** 2'

But Pint also extends the standard formatting capabilities for unicode and latex representations::

   >>> accel = 1.3 * ureg['meter/second**2']
   >>> 'The pretty representation is {:!p}'.format(accel) # Pretty print
   'The pretty representation is 1.3 meter/second²'
   >>> 'The latex representation is {:!l}'.format(accel) # Latex print
   'The latex representation is 1.3 \\frac{meter}{second^{2}}'

If you want to use abbreviated unit names, suffix the specification with `~`:

   >>> 'The str is {:!s~}'.format(accel)
   'The str is 1.3 m / s ** 2'

The same is true for repr (`r`), latex (`l`) and pretty (`p`) specs.


Using it in your projects
-------------------------

If you use Pint in multiple modules within you Python package, you normally want to avoid creating multiple instances of the unit registry.
The best way to do this is by instantiating the registry in a single place. For example,`you can add the following code to your package `__init__.py`::

   from pint import UnitRegistry
   Q_ = UnitRegistry().Quantity

Then in `yourmodule.py` the code would be::

   from . import Q_

   my_speed = Quantity(20, 'm/s')


