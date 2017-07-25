Types
-----

Most :ref:`types <syntax-type>` are universally valid.
However, restrictions apply to :ref:`limits <syntax-limits>`, which must be checked during validation.
Moreover, :ref:`block types <syntax-blocktype>` are converted to plain :ref:`function types <syntax-functype>` for ease of processing.


.. index:: block type
   pair: validation; block type
   single: abstract syntax; block type
.. _valid-blocktype:

Block Types
~~~~~~~~~~~

:ref:`Block types <syntax-blocktype>` may be expressed in one of two forms, both of which are converted to plain :ref:`function types <syntax-functype>` by the following rules.

:math:`\typeidx`
................

* The type :math:`C.\CTYPES[\typeidx]` must be defined in the context.

* Then the block type is valid as :ref:`function type <syntax-functype>` :math:`C.\CTYPES[\typeidx]`.

.. math::
   \frac{
     C.\CTYPES[\typeidx] = \functype
   }{
     C \vdash \typeidx : \functype
   }


:math:`[\valtype^?]`
....................

* The block type is valid as :ref:`function type <syntax-functype>` :math:`[] \to [\valtype^?]`.

.. math::
   \frac{
   }{
     C \vdash [\valtype^?] : [] \to [\valtype^?]
   }



.. index:: limits
   pair: validation; limits
   single: abstract syntax; limits
.. _valid-limits:

Limits
~~~~~~

:ref:`Limits <syntax-limits>` must have menaingful bounds.

:math:`\{ \LMIN~n, \LMAX~m^? \}`
................................

* If the maximum :math:`m^?` is not empty, then its value must not be smaller than :math:`n`.

* Then the limit is valid.

.. math::
   \frac{
     (n \leq m)^?
   }{
     \vdash \{ \LMIN~n, \LMAX~m^? \} \ok
   }
