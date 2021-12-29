
**Contents**


* `ImageStore Module <#imagestore-module>`__

  * `Configuration <#configuration>`__

    * `Detailed description on each of the keys used <#detailed-description-on-each-of-the-keys-used>`__

ImageStore Module
=================

The Image Store component of EII comes as a separate container which primarily
subscribes to the stream that comes out of the VideoAnalytics app via EII
MessageBus and stores the frame into minio for historical analysis.

The high level logical flow of ImageStore is as below:


#. The messagebus subscriber in ImageStore will subscribe to the VideoAnalytics
   published classified result (metadata, frame) on the messagebus.
   The img_handle is extracted out of the metadata and is used as the key and
   the frame is stored as a value for that key in minio persistent storage.
#. For historical analysis of the stored classified images, ImageStore starts
   the messagebus server which provides the read and store interfaces.
   The payload format is as follows for:

   * Store interface:
     .. code-block::

           Request: map ("command": "store","img_handle":"$handle_name"),[]byte($binaryImage)
           Response : map ("img_handle":"$handle_name", "error":"$error_msg") ("error" is optional and available only in case of error in execution.)

   * Read interface:
     .. code-block::

           Request : map ("command": "read", "img_handle":"$handle_name")
           Response : map ("img_handle":"$handle_name", "error":"$error_msg"),[]byte($binaryImage) ("error" is optional and available only in case of error in execution. And $binaryImage is available only in case of successful read)

Configuration
-------------

All the ImageStore module configuration are added into etcd (distributed
key-value data store) under ``AppName`` as mentioned in the
environment section of this app's service definition in docker-compose.

If ``AppName`` is ``ImageStore``\ , then the app's config would look like as below
 for ``/ImageStore/config`` key in Etcd:

.. code-block::

       "/ImageStore/config": {
           "minio":{
              "accessKey":"admin",
              "secretKey":"password",
              "retentionTime":"1h",
              "retentionPollInterval":"60s",
              "ssl":"false"
           }
       }

Detailed description on each of the keys used
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. list-table::
   :header-rows: 1

   * - Key
     - Description
     - Possible Values
     - Required/Optional
   * - accessKey
     - Username required to access Minio DB
     - Any suitable value
     - Required
   * - secretKey
     - Password required to access Minio DB
     - Any suitable value
     - Required
   * - retentionTime
     - The retention parameter specifies the retention policy to apply for the images stored in Minio DB.  In case of infinite retention time, set it to "-1"
     - Suitable duration string value as mentioned at https://golang.org/pkg/time/#ParseDuration.
     - Required
   * - retentionPollInterval
     - Used to set the time interval for checking images for expiration. Expired images will become candidates for deletion and no longer retained. In case of infinite retention time, this attribute will be ignored
     - Suitable duration string value as mentioned at https://golang.org/pkg/time/#ParseDuration
     - Required
   * - ssl
     - If "true", establishes a secure connection with Minio DB else a non-secure connection
     - "true" or "false"
     - Required


For more details on Etcd secrets and messagebus endpoint configuration, visit `Etcd_Secrets_Configuration.md <https://github.com/open-edge-insights/eii-core/blob/master/Etcd_Secrets_Configuration.md>`_ and
`MessageBus Configuration <https://github.com/open-edge-insights/eii-core/blob/master/common/libs/ConfigMgr/README.md#interfaces>`_ respectively.
