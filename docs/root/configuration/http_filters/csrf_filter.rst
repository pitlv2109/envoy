.. _config_http_filters_csrf:

CSRF
====

This is a filter which prevents Cross-Site Request Forgery based on a route or virtual host settings.
At it's simplest, CSRF is an attack that occurs when a malicious third-party
exploits a vulnerability that allows them to submit an undesired request on the
user's behalf.

A real-life example is cited in section 1 of `Robust Defenses for Cross-Site Request Forgery <https://seclab.stanford.edu/websec/csrf/csrf.pdf>`_:

    "For example, in late 2007 [42], Gmail had a CSRF vulnerability. When a Gmail user visited
    a malicious site, the malicious site could generate a request to Gmail that Gmail treated
    as part of its ongoing session with the victim. In November 2007, a web attacker exploited
    this CSRF vulnerability to inject an email filter into David Airey’s Gmail account [1]."

There are many ways to mitigate CSRF, some of which have been outlined in the
`OWASP Prevention Cheat Sheet <https://github.com/OWASP/CheatSheetSeries/blob/5a1044e38778b42a19c6adbb4dfef7a0fb071099/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.md>`_.
This filter employs a stateless mitigation pattern known as origin verification.

This pattern relies on two pieces of information used in determining if
a request originated from the same host.
* The origin that caused the user agent to issue the request (source origin).
* The origin that the request is going to (target origin).

When the filter is evaluating a request, it ensures both pieces of information are present
and compares their values. If the source origin is missing or the origins do not match
the request is rejected.

  .. note::
    Due to differing functionality between browsers this filter will determine
    a request's source origin from the Host header. If that is not present it will
    fall back to the host and port value from the requests Referer header.


For more information on CSRF please refer to the pages below.

* https://www.owasp.org/index.php/Cross-Site_Request_Forgery_%28CSRF%29
* https://seclab.stanford.edu/websec/csrf/csrf.pdf
* :ref:`v2 API reference <envoy_api_msg_config.filter.http.csrf.v2.CsrfPolicy>`

  .. note::

    This filter should be configured with the name *envoy.csrf*.

.. _csrf-runtime:

Runtime
-------

The CSRF filter supports the following RuntimeFractionalPercent settings:

filter_enabled
  The % of requests for which the filter is enabled. The default is
  100/:ref:`HUNDRED <envoy_api_enum_type.FractionalPercent.DenominatorType>`.

  To utilize runtime to enabled/disable the CSRF filter set the
  :ref:`runtime_key <envoy_api_msg_core.runtimefractionalpercent>`
  value of the :ref:`filter_enabled <envoy_api_msg_config.filter.http.csrf.v2.CsrfPolicy>`
  field.

shadow_enabled
  The % of requests for which the filter is enabled in shadow only mode. Default is 0.
  If present, this will evaluate a request's *Origin* and *Destination* to determine
  if the request is valid but will not enforce any policies.

  To utilize runtime to enabled/disable the CSRF filter's shadow mode set the
  :ref:`runtime_key <envoy_api_msg_core.runtimefractionalpercent>`
  value of the :ref:`shadow_enabled <envoy_api_msg_config.filter.http.csrf.v2.CsrfPolicy>`
  field.

To determine if the filter and/or shadow mode are enabled you can check the runtime
values via the admin panel at :http:get:`/runtime`.

.. note::

  If both ``filter_enabled`` and ``shadow_enabled`` are on, the ``filter_enabled``
  flag will take precedence.

.. _csrf-statistics:

Statistics
----------

The CSRF filter outputs statistics in the <stat_prefix>.csrf.* namespace.

.. csv-table::
  :header: Name, Type, Description
  :widths: 1, 1, 2

  missing_source_origin, Counter, Number of requests that are missing a source origin header.
  request_invalid, Counter, Number of requests whose source and target origins do not match.
  request_valid, Counter, Number of requests whose source and target origins match.
