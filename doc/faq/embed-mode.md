---
title: Embed mode
layout: page
faq: true
categories: [Integrations, Customisation]
---

If the ``embed=1`` URL parameter is used, the client runs in embed mode and the page will send a "ready" message to the opener or parent when the page is loaded. After receiving the ready message, diagram data can be sent as XML or compressed XML. It will return XML or an empty string if _Apply_ or _Cancel_ are selected, respectively. The following URL parameters are available in embed mode:
* ``spin=1``: Shows a _Loading..._ spinner while waiting for the diagram data to load in embed mode.
* ``modified=0``: Disables the update of the modified state in embed mode after _Save_ is selected and enables the update of the status message after changes are made. If ``0`` is used, the status bar is cleared after the changes. Otherwise, the value of this is used as the resource key. Instead of using this URL parameter, you can specify this setting in the load message.
* ``keepmodified=1``: If ``modified`` specifies a resource key, this is used to maintain the modified state after _Save_ is selected.
* ``libraries=1``: Specifies whether libraries should be enabled in embed mode. The default is disabled (``0``).
* ``saveAndExit``: Displays an additional _Save and Exit_ button. Instead of using this URL parameter, you can specify this setting in the load message.
* ``ready=message``: Specifies the message to send in embed or client mode. Default is ``'ready'``. If the JSON protocol is used, then this is ignored.
* ``returnbounds=1``: Returns a JSON structure with the diagram bounds in embed and client mode. This message is dispatched immediately after receiving the diagram XML.
* ``proto=json``: Uses JSON for message passing in embed and client mode.
* ``configure=1``: Sends the ``configure`` event and waits for the ``configure`` action.

## Configuration

If the ``configure=1`` URL parameter is used, the client sends ``{event: 'configure'}`` and waits for ``{action: 'configure', config: obj}`` before creating the main application and sending the ``init`` event. The ``obj`` is [specified in the configuration](/doc/faq/diagram-editor-configuration.html), e.g. ``config: {defaultFonts: ["Humor Sans"]}``.


## Description of the JSON protocol

When the client is ready, it sends ``{event: 'init'}`` and expects ``{action: 'load', xml: '...'}``. Specify an optional ``autosave: 1`` in this message to enable the autosave feature.

You can set the optional ``modified`` flag using the same semantic as the ``modified`` URL parameter which is defined above. The optional ``saveAndExit`` flag can be specified with the same semantic as the ``saveAndExit`` URL parameter which is defined above. The URL parameters have precedence over the flags set in this way.

Specify the optional ``title`` string to show a title in the menu bar (on the right).

If ``autosave`` is enabled the editor will send the current XML in an ``{event: 'autosave'...}``.

The XML in the load message is displayed and an ``{event: 'load'...}`` is returned with some data about the size of the diagram. The XML in this message can be any supported XML representation of the diagram including SVG or [PNG files with embedded XML](/blog/xml-in-png.html). The old parameter name to pass PNG+XML ``xmlpng`` can still be used.

**Note:** For XML, all supported file formats can be used, including PNG+XML, the data part of an SVG data URI with UTF8 encoding, or the complete SVG or PNG data URI with base64 encoding. For all .vsd* files, a data URI with a
``data:application/vnd.visio;base64`` prefix must be used. Lucidchart and Gliffy files are represented as a JSON string .

When you select _Save_, the same message as the ``load`` event is sent, but with an additional event: ``'save'`` and ``xml: '...'`` containing the XML of the diagram. When you select _Save and Exit_, the message includes an additional ``exit: true``. If you select _Exit_ , an ``{event: 'exit', modified: boolean}`` is sent.

* ``{action: 'dialog', title: '...', message: '...', button: '...', modified: bool}`` can be sent at any time to display a dialog in the editor window. To translate the dialog, use ``titleKey``, ``messageKey`` and ``buttonKey`` instead.
* ``{action: 'prompt', title: '...', okKey: '...', defaultValue: '...'}`` can be sent at any time to display a prompt in the editor window. To translate the title, use ``titleKey`` instead. When _OK_ is selected, an ``{event: 'prompt', value: '...', message: ...}`` with the incoming message and the value entered in the dialog is returned.
* ``{action: 'template'}`` can be sent at any time to show the template dialog. This is normally sent instead of a ``load`` message to create an initial diagram. When _Create_ is selected, the diagram is created from the selected template. If _Cancel_ is selected, an ``exit`` message is sent. Specify an optional ``callback: true`` parameter to pass the current template and filename back to the caller for validation. The message for the callback is ``{event: 'template', xml: '...', blank: '...', name: ...}`` where ``blank`` is ``true`` if a blank diagram was selected in the template dialog.
* ``{action: 'draft', xml: '...', name: '...', editKey: '...', discardKey: '...', ignore: bool}`` can be sent to show a draft dialog. If _Edit_ or _Discard_ is selected, an ``{event: 'draft', error/result: '...', message: ...}`` with the incoming message and the result of _'Edit'_ or _'Discard'_ is returned. If there was an error when displaying the dialog, the ``error`` is returned instead of the ``result`` in the message. If ``ignore`` is true, an ``ignore`` option is used which returns ``result: 'ignore'``.
* ``{action: 'status', message: '...', modified: bool}`` can be sent at any time to display a message in the status bar. The optional ``modified`` flag is used to update the modified state. Instead of a ``message``, you can use ``messageKey`` to specify a resource key for the message.
* ``{action: 'spinner', message: '...', show: bool, enabled: bool}`` can be sent at any time to display a spinner with ``message`` or hide the current spinner if ``show`` is set to ``false``. Instead of a ``message``, you can use ``messageKey`` to specify a resource key for the message.
* ``{action: 'export', format: '...'}`` can be sent at any time to return ``{event: 'export', format: '...', message: ..., data: '...', xml: '...'}`` with the incoming message, data and XML used for the given format. Supported formats include ``html`` (former embed format), ``html2`` (new embed format), ``svg`` (default), ``xmlsvg`` (SVG with embeddded XML), ``png`` and ``xmlpng`` (PNG with embedded XML). The ``data`` parameter in the ``export`` event contains a valid data URI for the given export format. Currently, a base64 encoding is used, but this may change in the future.
   * For ``png`` and ``xmlpng``, use an additional ``spin`` (or ``spinKey``) parameter to enable a spinner and specify a ``message`` (or ``messageKey``) while the image is being generated.
   * Use the optional ``xml`` parameter to specify the XML of the diagram to be exported for all supported formats.
   * Use the optional ``embedImages: false`` for the ``svg`` and ``xmlsvg`` formats to disable embedded images in the SVG output.

If any unknown message is received, the system responds with ``{error: 'unknownMessage'}``.

### Examples

* [Introduction and example](https://github.com/jgraph/drawio-html5)
* [WebDAV example](https://github.com/jgraph/drawio-webdav)
* [GitHub Example](https://github.com/jgraph/drawio-github)
