==========
Data Types
==========

Presto has a set of built-in data types, described below.
Additional types can be provided by plugins.

.. note::

    Connectors are not required to support all types.
    See connector documentation for details on supported types.

.. contents::
    :local:
    :backlinks: none
    :depth: 1

Boolean
-------

``BOOLEAN``
^^^^^^^^^^^

This type captures boolean values ``true`` and ``false``.

Integer
-------

.. note::

    Cast conversion functions allow leading and trailing spaces when casting
    a string to ``TINYINT``, ``SMALLINT``, ``INTEGER``, or ``BIGINT``.

``TINYINT``
^^^^^^^^^^^

A 8-bit signed two's complement integer with a minimum value of
``-2^7`` and a maximum value of ``2^7 - 1``.

``SMALLINT``
^^^^^^^^^^^^

A 16-bit signed two's complement integer with a minimum value of
``-2^15`` and a maximum value of ``2^15 - 1``.

``INTEGER``
^^^^^^^^^^^

A 32-bit signed two's complement integer with a minimum value of
``-2^31`` and a maximum value of ``2^31 - 1``.  The name ``INT`` is
also available for this type.

``BIGINT``
^^^^^^^^^^

A 64-bit signed two's complement integer with a minimum value of
``-2^63`` and a maximum value of ``2^63 - 1``.

Floating-Point
--------------

``REAL``
^^^^^^^^

A real is a 32-bit inexact, variable-precision implementing the
IEEE Standard 754 for Binary Floating-Point Arithmetic.

Presto strays from the IEEE standard when handling NaNs.
In Presto, NaN is considered larger than any other value for
all comparison and sorting operations. Additionally, NaN=NaN will
be true for all equality and distinctness purposes.

``DOUBLE``
^^^^^^^^^^

A double is a 64-bit inexact, variable-precision implementing the
IEEE Standard 754 for Binary Floating-Point Arithmetic.

Presto strays from the IEEE standard when handling NaNs.
In Presto, NaN is considered larger than any other value for
all comparison and sorting operations. Additionally, NaN=NaN will
be true for all equality and distinctness purposes.

Fixed-Precision
---------------

``DECIMAL``
^^^^^^^^^^^

A fixed precision decimal number. Precision up to 38 digits is supported
but performance is best up to 18 digits.

The decimal type takes two literal parameters:

- **precision** - total number of digits

- **scale** - number of digits in fractional part. Scale is optional and defaults to 0.

Example type definitions: ``DECIMAL(10,3)``, ``DECIMAL(20)``

Example literals: ``DECIMAL '10.3'``, ``DECIMAL '1234567890'``, ``1.1``

.. note::

    Decimal literals without explicit type specifier such as ``1.2``
    are parsed as values of the ``DECIMAL`` type by default.
    To change the behavior and parse such values as ``DOUBLE`` type set
    the following property to ``true``:

    - System wide property: ``parse-decimal-literals-as-double``
    - Session wide property: ``parse_decimal_literals_as_double``

String
------

``VARCHAR``
^^^^^^^^^^^

Variable length character data with an optional maximum length indicating
the maximum number of characters allowed.

Example type definitions: ``varchar``, ``varchar(20)``.

For ``varchar(20)`` the length value indicates that the string data may
consist of up to 20 characters.

SQL supports simple and Unicode string literals:
 - Literal string : ``'Hello winter !'``
 - Unicode string with default escape character: ``U&'Hello winter \2603 !'``
 - Unicode string with custom escape character: ``U&'Hello winter #2603 !' UESCAPE '#'``

A Unicode string is prefixed with ``U&`` and requires an escape character
before any Unicode character usage with 4 digits. In these examples
``\2603`` and ``#2603`` represent a snowman character. Long Unicode codes
with 6 digits require a plus symbol ``+`` before the code. For example,
use ``\+01F600`` for a grinning face emoji.

Single quotes in string literals can be escaped by using another single quote: ``'It''s a beautiful day!'``

``CHAR``
^^^^^^^^

Fixed length character data. A `CHAR` type without length specified has a
default length of 1. A `CHAR(x)` value always has `x` characters. For example,
casting `dog` to `CHAR(7)` adds 4 implicit trailing spaces. Leading and trailing
spaces are included in comparisons of `CHAR` values. As a result, two character
values with different lengths (`CHAR(x)` and `CHAR(y)` where `x != y`) are never
equal, but comparison of such values implicitly converts the types to the same
length and pads with spaces so that the following query returns `true`:

``SELECT cast('example' AS char(20)) = cast('example    ' AS char(25));``

As with `VARCHAR`, a single quote in a `CHAR`
literal can be escaped with another single quote:

``SELECT CHAR 'All right, Mr. DeMille, I''m ready for my close-up.'``


``VARBINARY``
^^^^^^^^^^^^^

Variable length binary data.

.. note::

    Binary strings with length are not yet supported: ``varbinary(n)``

``JSON``
^^^^^^^^

JSON value type, which can be a JSON object, a JSON array, a JSON number, a JSON string,
``true``, ``false`` or ``null``.

Date and Time
-------------

``DATE``
^^^^^^^^

Calendar date (year, month, day).

Example: ``DATE '2001-08-22'``

``TIME``
^^^^^^^^

Time of day (hour, minute, second, millisecond) without a time zone.
Values of this type are parsed and rendered in the session time zone.

Example: ``TIME '01:02:03.456'``

``TIME WITH TIME ZONE``
^^^^^^^^^^^^^^^^^^^^^^^

Time of day (hour, minute, second, millisecond) with a time zone.
Values of this type are rendered using the time zone from the value.

Example: ``TIME '01:02:03.456 America/Los_Angeles'``

``TIMESTAMP``
^^^^^^^^^^^^^

Instant in time that includes the date and time of day without a time zone.
Values of this type are parsed and rendered in the session time zone.

Example: ``TIMESTAMP '2001-08-22 03:04:05.321'``

``TIMESTAMP WITH TIME ZONE``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Instant in time that includes the date and time of day with a time zone.
Values of this type are rendered using the time zone from the value.

Example: ``TIMESTAMP '2001-08-22 03:04:05.321 America/Los_Angeles'``

``INTERVAL YEAR TO MONTH``
^^^^^^^^^^^^^^^^^^^^^^^^^^

Span of years and months.

Example: ``INTERVAL '3' MONTH``

``INTERVAL DAY TO SECOND``
^^^^^^^^^^^^^^^^^^^^^^^^^^

Span of days, hours, minutes, seconds and milliseconds.

Example: ``INTERVAL '2' DAY``

Structural
----------

.. _array_type:

``ARRAY``
^^^^^^^^^

An array of the given component type.

Example: ``ARRAY[1, 2, 3]``

.. _map_type:

``MAP``
^^^^^^^

A map between the given component types.

Example: ``MAP(ARRAY['foo', 'bar'], ARRAY[1, 2])``

.. _row_type:

``ROW``
^^^^^^^

A structure made up of named fields. The fields may be of any SQL type, and are
accessed with the field reference operator ``.``

Example: ``CAST(ROW(1, 2.0) AS ROW(x BIGINT, y DOUBLE))``

Network Address
---------------

.. _ipaddress_type:

``IPADDRESS``
^^^^^^^^^^^^^

An IP address that can represent either an IPv4 or IPv6 address.

Internally, the type is a pure IPv6 address. Support for IPv4 is handled
using the *IPv4-mapped IPv6 address* range (:rfc:`4291#section-2.5.5.2`).
When creating an ``IPADDRESS``, IPv4 addresses will be mapped into that range.

When formatting an ``IPADDRESS``, any address within the mapped range will
be formatted as an IPv4 address. Other addresses will be formatted as IPv6
using the canonical format defined in :rfc:`5952`.

Examples: ``IPADDRESS '10.0.0.1'``, ``IPADDRESS '2001:db8::1'``

.. _ipprefix_type:

``IPPREFIX``
^^^^^^^^^^^^

An IP routing prefix that can represent either an IPv4 or IPv6 address.

Internally, an address is a pure IPv6 address. Support for IPv4 is handled
using the *IPv4-mapped IPv6 address* range (:rfc:`4291#section-2.5.5.2`).
When creating an ``IPPREFIX``, IPv4 addresses will be mapped into that range.
Additionally, addresses will be reduced to the first address of a network.

``IPPREFIX`` values will be formatted in CIDR notation, written as an IP
address, a slash ('/') character, and the bit-length of the prefix. Any
address within the IPv4-mapped IPv6 address range will be formatted as an
IPv4 address. Other addresses will be formatted as IPv6 using the canonical
format defined in :rfc:`5952`.

Examples: ``IPPREFIX '10.0.1.0/24'``, ``IPPREFIX '2001:db8::/48'``

UUID
----

.. _uuid_type:

``UUID``
^^^^^^^^

This type represents a UUID (Universally Unique IDentifier), also known as a
GUID (Globally Unique IDentifier), using the format defined in :rfc:`4122`.

Example: ``UUID '12151fd2-7586-11e9-8f9e-2a86e4085a59'``

HyperLogLog
-----------

Calculating the approximate distinct count can be done much more cheaply than an exact count using the
`HyperLogLog <https://en.wikipedia.org/wiki/HyperLogLog>`__ data sketch. See :doc:`/functions/hyperloglog`.

.. _hyperloglog_type:

``HyperLogLog``
^^^^^^^^^^^^^^^

A HyperLogLog sketch allows efficient computation of :func:`!approx_distinct`. It starts as a
sparse representation, switching to a dense representation when it becomes more efficient.

.. _p4hyperloglog_type:

``P4HyperLogLog``
^^^^^^^^^^^^^^^^^

A P4HyperLogLog sketch is similar to :ref:`hyperloglog_type`, but it starts (and remains)
in the dense representation.

KHyperLogLog
------------

.. _khyperloglog_type:

``KHyperLogLog``
^^^^^^^^^^^^^^^^

A KHyperLogLog is a data sketch that can be used to compactly represents the association of two
columns. See :doc:`/functions/khyperloglog`.

SetDigest
---------

.. _setdigest_type:

``SetDigest``
^^^^^^^^^^^^^

A SetDigest (setdigest) is a data sketch structure used
in calculating `Jaccard similarity coefficient <https://wikipedia.org/wiki/Jaccard_index>`_
between two sets.

SetDigest encapsulates the following components:

- `HyperLogLog <https://wikipedia.org/wiki/HyperLogLog>`_
- `MinHash with a single hash function <http://wikipedia.org/wiki/MinHash#Variant_with_a_single_hash_function>`_

The HyperLogLog structure is used for the approximation of the distinct elements
in the original set.

The MinHash structure is used to store a low memory footprint signature of the original set.
The similarity of any two sets is estimated by comparing their signatures.

SetDigests are additive, meaning they can be merged together.

SFM Sketch
-----------

.. _sfmsketch_type:

``SfmSketch``
^^^^^^^^^^^^^

The Sketch-Flip-Merge (SFM) data sketch is a noisy, random distinct-counting
sketch similar to :ref:`hyperloglog_type`. See :func:`!noisy_approx_set_sfm`.

Quantile Digest
---------------

.. _qdigest_type:

``QDigest``
^^^^^^^^^^^

A quantile digest (qdigest) is a summary structure which captures the approximate
distribution of data for a given input set, and can be queried to retrieve approximate
quantile values from the distribution.  The level of accuracy for a qdigest
is tunable, allowing for more precise results at the expense of space.

A qdigest can be used to give approximate answer to queries asking for what value
belongs at a certain quantile.  A useful property of qdigests is that they are
additive, meaning they can be merged together without losing precision.

A qdigest may be helpful whenever the partial results of ``approx_percentile``
can be reused.  For example, one may be interested in a daily reading of the 99th
percentile values that are read over the course of a week.  Instead of calculating
the past week of data with ``approx_percentile``, ``qdigest``\ s could be stored
daily, and quickly merged to retrieve the 99th percentile value.

See :doc:`/functions/qdigest`.

T-Digest
--------

.. _tdigest_type:

``TDigest``
^^^^^^^^^^^

A t-digest is similar to :ref:`qdigest <qdigest_type>`, but it uses `a different algorithm
<http://dx.doi.org/10.1145/347090.347195>`_ to represent the approximate distribution of a set
of numbers. T-digest has better performance than quantile digests but only supports the
``DOUBLE`` type. See :doc:`/functions/tdigest`.

KLL Sketch
----------

.. _kll_sketch_type:

``KLL Sketch``
^^^^^^^^^^^^^^

A KLL sketch is similar to the :ref:`qdigest <qdigest_type>`, but, like the
T-Digest uses a `different algorithm
<https://datasketches.apache.org/docs/KLL/KLLSketch.html>`_ to represent the
approximate distribution of a set of values. The KLL sketch in Presto
supports int, bigint, double, varchar, and boolean types. See
:doc:`/functions/sketch` for more information. In serialized form, the
``kllsketch`` type stored by Presto can be read directly by any other
application which utilizes the Apache DataSketches library to read KLL
sketches.

Geospatial
----------

.. _geospatial_type:

``Geospatial``
^^^^^^^^^^^^^^

Geospatial types in Presto are designed to handle and analyze spatial data efficiently,
adhering to the SQL/MM specification and the Open Geospatial Consortium standards.
These types include ``POINT``, ``LINESTRING``, ``POLYGON``, ``MULTIPOINT``, ``MULTILINESTRING``, ``MULTIPOLYGON``,
and ``GEOMETRYCOLLECTION``, which can be expressed in Well-Known Text (WKT) and Well-Known Binary (WKB) formats.
The types support operations such as spatial measurements and relationship checks,
crucial for geographic information systems (GIS) and other applications requiring spatial data manipulation.
The geospatial types ensure data integrity and provide robust tools for complex spatial querying and analysis.

See :doc:`/functions/geospatial`.
