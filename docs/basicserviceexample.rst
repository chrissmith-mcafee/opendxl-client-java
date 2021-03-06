Basic Service Sample
=====================

This sample demonstrates how to register a DXL service to receive `Request <javadoc/index.html?com/opendxl/client/message/Request.html>`_
messages and send `Response <javadoc/index.html?com/opendxl/client/message/Response.html>`_ messages back to an invoking
`DxlClient <javadoc/index.html?com/opendxl/client/DxlClient.html>`_.

Prior to running this sample make sure you have completed the samples configuration step (:doc:`sampleconfig`).

To run this sample execute the ``sample\runsample`` script as follows:

    .. parsed-literal::

        c:\\dxlclient-java-sdk-\ |version|\>sample\\runsample sample.basic.ServiceSample

The output should appear similar to the following:

    .. code-block:: python

        Service received request payload: ping
        Client received response payload: pong

The code for the sample is broken into two main sections.

The first section is responsible for creating a `RequestCallback <javadoc/index.html?com/opendxl/client/callback/RequestCallback.html>`_ that will be
invoked for a specific topic associated with the service. The callback will send back a
messages and send `Response <javadoc/index.html?com/opendxl/client/message/Response.html>`_ message with a payload of ``pong`` for any
`Request <javadoc/index.html?com/opendxl/client/message/Request.html>`_ messages that are received.

It then creates a `ServiceRegistrationInfo <javadoc/index.html?com/opendxl/client/ServiceRegistrationInfo.html>`_ instance and registers the request
callback with it via the `ServiceRegistrationInfo.addTopic() <javadoc/com/opendxl/client/ServiceRegistrationInfo.html#addTopic-java.lang.String-com.opendxl.client.callback.RequestCallback->`_ method.

Finally it registers the service with the fabric via the `DxlClient.registerServiceSync() <javadoc/com/opendxl/client/DxlClient.html#registerServiceSync-com.opendxl.client.ServiceRegistrationInfo-long->`_
method of the `DxlClient <javadoc/index.html?com/opendxl/client/DxlClient.html>`_.

    .. code-block:: java

        //
        // Register the service
        //

        // Create incoming request callback
        final RequestCallback myRequestCallback =
            request -> {
                try {
                    System.out.println("Service received request payload: "
                        + new String(request.getPayload(), Message.CHARSET_UTF8));

                    final Response res = new Response(request);
                    res.setPayload("pong".getBytes(Message.CHARSET_UTF8));

                    client.sendResponse(res);
                } catch (Exception ex) {
                    ex.printStackTrace();
                }
            };

        // Create service registration object
        final ServiceRegistrationInfo info = new ServiceRegistrationInfo(client, "myService");

        // Add a topic for the service to respond to
        info.addTopic(SERVICE_TOPIC, myRequestCallback);

        // Register the service with the fabric (wait up to 10 seconds for registration to complete)
        client.registerServiceSync(info, TIMEOUT);

The second section sends a `Request <javadoc/index.html?com/opendxl/client/message/Request.html>`_ message to the service
that contains a payload of ``ping`` via the `DxlClient.syncRequest() <javadoc/com/opendxl/client/DxlClient.html#syncRequest-com.opendxl.client.message.Request-long->`_
method of the `DxlClient <javadoc/index.html?com/opendxl/client/DxlClient.html>`_.

The payloads of the `Request <javadoc/index.html?com/opendxl/client/message/Request.html>`_ and
`Response <javadoc/index.html?com/opendxl/client/message/Response.html>`_ messages are printed.

    .. code-block:: java

        //
        // Invoke the service (send a request)
        //

        // Create the request message
        final Request req = new Request(SERVICE_TOPIC);

        // Populate the request payload
        req.setPayload("ping".getBytes(Message.CHARSET_UTF8));

        // Send the request and wait for a response (synchronous)
        final Response res = client.syncRequest(req);

        // Extract information from the response (Check for errors)
        if (res.getMessageType() != Message.MESSAGE_TYPE_ERROR) {
            System.out.println("Client received response payload: "
                + new String(res.getPayload(), Message.CHARSET_UTF8));
        } else {
            System.out.println("Error: " + ((ErrorResponse) res).getErrorMessage());
        }
