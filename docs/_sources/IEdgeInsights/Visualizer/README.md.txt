# Intel Edge Insights Simple Visualizer
Simple visualizer for the EII platform.

### 1. Running as a docker container

#### Steps to build and run viualizer

* Follow [provision/README.md](../README#provision.md) for Provisioning
  if not done already as part of EII stack setup

* Running visualizer as a container from [build](../../build):

  ```sh
  $ xhost +
  $ docker-compose up --build ia_visualizer
  ```

-----
**NOTE**:
1. The admin has to make sure all the necessary config is set in etcd before starting the visualizer.
2. The user has to make sure the path provided in docker-compose volumes of visualizer correlates to the one in etcd before running visualizer if he wishes to save images.
3. Run this command in terminal if you run into tkinter couldn't connect to display exception

   ```sh
   $ xhost +
   ```
4. If the Visualizer UI doesn’t show up and if you notice couldn't connect to display ":0" error
   in `docker logs -f ia_visualizer`, please check the value for `DISPLAY` env variable on the host
   machine by running cmd: `env | grep DISPLAY`, please set this as the value for the `DISPLAY`
   nv variable in the ia_visualizer service of [docker-compose.yml](docker-compose.yml) or in the
   consolidated [../build/docker-compose.yml](../build/docker-compose.yml) file and re-run
   `docker-compose up ia_visualizer -d`

    Example:
    ```sh
    $ env | grep DISPLAY
    DISPLAY:=1
    ```
    Set “:=1” as `DISPLAY` env value in ia_visualizer service
-----

* If one needs to remove the classified images on a periodic basis:

  1. Have this command running in a separate terminal as a cleanup task to remove images older than 60 mins in IMAGE_DIR. Replace <path-to-IMAGE_DIR> with IMAGE_DIR path given while running visualizer. The -mmin option can be changed accordingly by the user.

    ```sh
    $ while true; do find <path-to-IMAGE_DIR> -mmin +60 -type f -name "*.png" -exec rm -f {} \;;  done
    ```
* If user needs to remove the bounding box:

  1. Set the value of draw_results in config.json as false for both Visualiser and WebVisualiser.

    ```
    draw_results: "false"
    ```
* If user needs to save images of visualizer:
   
  1. Set the value of save_image in config.json as true 

     ```
    "save_image": "true"
    ``` 
   
#### Using Labels

  In order to have the visualizer label each of the defects on the image, labels in JSON format(with mapping between topic subscribed text to be displayed) has to be provided in [config.json](./config.json) file and run the [builder.py](../build/builder.py) script using the below command.
  ```sh
  $ python3 builder.py
  ```

  An example of what this JSON value should look like is shown below. In this case
  it is assumed that the classification types are `0` and `1` and the text labels
  to be displayed are `MISSING` and `SHORT` respectively.

  ```json
  {
      "0": "MISSING",
      "1": "SHORT"
  }
  ```
  > **NOTE:** These labels are the mapping for the PCB demo provided in EII's visualizer directory. Currently camera1_stream_results consists of pcb demo labeling and camera2_stream_results consists of safety demo labeling.
  Hence, in [config.json](./config.json) proper mapping of all the subscribed topics should be done with pcb demo labeling and safety demo labeling respectively.

```json
  "/Visualizer/config": {
      "save_image": "false",
      "draw_results": "false",
      "labels" : {
          "camera1_stream_results": {
              "0": "MISSING",
              "1": "SHORT"
          },
          "native_safety_gear_stream_results": {
               "1": "safety_helmet",
               "2": "safety_jacket",
               "3": "Safe",
               "4": "Violation"
          },
          "py_safety_gear_stream_results": {
               "1": "safety_helmet",
               "2": "safety_jacket",
               "3": "Safe",
               "4": "Violation"

          },
         "gva_safety_gear_stream_results": {
               "1": "safety_helmet",
               "2": "safety_jacket",
               "3": "Safe",
               "4": "Violation"
          }
      }
  }
```

### Metadata Structure

EII Visualizer app can decode certain types of mete-data formats for drawing the defects on the image.
Any application wanting to use EII visualizer need to comply with the meta-data format as described below:

A) For Ingestor's **Non-GVA** type, metadata structure sample is :

```json
{
 'channels': 3,
 'encoding_type': 'jpeg',
 'height': 1200,

 'defects': [
     {'type': 0, 'tl': [1019, 644], 'br': [1063, 700]},
     {'type': 0, 'tl': [1297, 758], 'br': [1349, 796]}
    ],

'display_info': [{'info':'good', 'priority':0}],

'img_handle': '348151d424',
'width': 1920,
'encoding_level': 95
}
```

where in `defects` and `display_info` is a list of dicts.

Each entry in `defects` list is a dictionary that should contain following keys:
* `type` : value given to type will be the label id
* `tl` : value is the top-left `x` and `y` co-ordinate of the defect in the image.
* `br` : value is the bottom-right `x` and `y` co-ordinate of the defect in the image.

Each entry in `display_info` list is a dictionary that should contain following keys:
* `info` : value given will be displayed on the image.
* `priority` : Based on the priority level (0, 1, or 2), info will be displayed in either green, orange or red.
    * 0 : Low priority, info will be displayed in green.
    * 1 : Medium priority, info will be displayed in orange.
    * 2 : High priority, info will be displayed in red.

----
B) For Ingestor's **GVA** type, metadata structure sample is :

```json
{
    'channels': 3,
    'gva_meta': [

        {'x': 1047, 'height': 86, 'y': 387, 'width': 105, 'tensor': [{'label': '', 'label_id': 1, 'confidence':0.8094226121902466, 'attribute':'detection'}]},

        {'x': 1009, 'height': 341, 'y': 530, 'width': 176, 'tensor': [{'label': '', 'label_id': 2, 'confidence': 0.9699158668518066, 'attribute': 'detection'}]}

        ],

    'encoding_type': 'jpeg',
    'height': 1080,
    'img_handle': '7247149a0d',
    'width': 1920,
    'encoding_level': 95
}

```
where in `gva_meta` is a list of dicts.

**NOTE**: Any data with in the list, tuple or dict of meta data should be of primitive data type (int, float, string, bool). Refer the examples given above.
