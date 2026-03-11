`GnssSensor` is a component which simulates the position of vehicle computed by the *Global Navigation Satellite System* based on the transformation of the *GameObject* to which this component is attached.
The `GnssSensor` allows you to select the output format: [*NavSatFix*](https://docs.ros.org/en/ros2_packages/humble/api/sensor_msgs/msg/NavSatFix.html) or the [*MGRS*](https://www.maptools.com/tutorials/mgrs/quick_guide) coordinate system and [*Geo coordinate system*](https://en.wikipedia.org/wiki/Geographic_coordinate_system).
It is also possible to enable Heading output.
The `GnssSensor` can be configured with sensor noise consisting of a Gauss-Markov process and a delay following a Gamma distribution.

## Prefab

```{.yml .no-copy}
Assets/Awsim/Prefabs/Entity/EgoVehicle/Sensor/Gnss/GnssSensor.prefab
```

<br>


## GnssSensor class

`GnssSensor` allows you to select the output format: NavSatFix or the MGRS coordinate system and Geo coordinate system. It is also possible to enable Heading output.
The sensor outputs data based on the configured period.
The `GnssSensor` can be configured with sensor noise consisting of a Gauss-Markov process and a delay following a Gamma distribution.

### prerequisites

`MgrsPosition` and `GeoCoordinatePosition` need to be set up. From these classes, the output is converted to each coordinate system by considering the Unity world coordinate system origin and sensor position.
The accuracy of the simulated delay depends on the TimeSource configuration. When configuring a delay, please select a `TimeSourceType` with sufficient precision.

### How to use

1. Setup `MgrsPosition` and `GeoCoordinatePosition` in the scene.
1. Initialize `GnssSensor`.


### Setting parameters

|Type|Parameter|Feature|
|:--|:--|:--|
|`int`|`_outputHz`|Period to output.|
|`GnssOutputMode`|`_outputMode`|Mgrs or NavSatFix.|
|`bool`|`_attitudeOutput`|Enable heading output.|
| `Vector3` | `ProcessVar` | Variance of the noise process [$m^2$]. |
| `Vector3` | `WhiteNoiseVar` | Variance of the random walk [$m^2$]. |

### Output data

It is contained in the `GnssSensor.IReadOnlyOutputData` type.

|Type|Parameter|Feature|
|:--|:--|:--|
|`Mgrs`|`Mgrs`|MGRS coordinate system position.|
|`GeoCoordinate`|`GeoCoordinate`|GeoCoordinate coordinate system position.|
|`AttEuler`|`AttEuler`|Attitude and angular velocity. Currently only Heading.|

### Geo reference

`GnssSensor` MGRS outputs the sum of the origin value set for each geo reference and the value obtained by converting the Unity world coordinates to each geo reference. There are two classes of georeferencing.

|Geo reference|Feature|
|:--|:--|
|`MgrsPosition`|Origin value of the Unity world coordinate system (0, 0, 0) transformed in the MGRS system.|
|`GeoCoordinatePosition`|Origin value of the Unity world coordinate system (0, 0, 0) transformed in the Geo coordinate system.|




### Add output callback

GnssSensor can add an `Action` type callback that takes `GnssSensor.IReadOnlyOutputData` as an argument.

```cs
// sample code from GnssRos2Publisher.cs
_gnssSensor.OnOutput += Publish;
```

<br> 

## GnssRos2Publisher class

`GnssRos2Publisher` converts the output of `GnssSensor` to ROS2 and publishes the topic.

### Setting parameters

|Type|Parameter|Feature|
|:--|:--|:--|
|`string`|`_poseTopic`|`geometry_msgs/Pose` msg topic name.|
|`string`|`_poseWithCovarianceStampedTopic`|`geometry_msgs/PoseWithCovarianceStamped` msg topic name.|
|`string`|`_mgrsFrame`|MGRS frame ID of ros2.|
|`string`|`_navSatFixTopic`|`sensor_msgs/NavSatFix` msg topic name.|
|`string`|`_frameID`|NavSatFix frame ID of ros2.|
|`string`|`_orientationTopic`|`autoware_sensing_msgs/GnssInsOrientationStamped` msg topic name.|
|`QosSettings`|`_qosSettings`|Quality of Service settings of ros2.|
|`GnssSensor`|`_gnssSensor`|Target `GnssSensor` instance.|
|`int`|`_highFreqUpdateHz`|Update period for publishing.<=1000|
|`bool`|`_gammaDelay`|Enable delay setting.|
|`float`|`_gammaDelayMeanMs`| Mean of the distribution (including bias). |
|`float`|`_gammaDelayVariance`| Variance of the distribution ($\text{milliseconds}^2$). |
|`float`|`_gammaDelayMinMs`| Minimum value of the distribution. |
|`float`|`_gammaDelayMaxMs`| Max value of the distribution. |

### Default publish topics

`GnssRos2Publisher` is configured by default with the following two topics publishing.

| Topic| Message type | `frame_id` | `Hz` | `QoS` |
|:---|:---|:---|:---:|:---|
| `/sensing/gnss/pose`                 | [`geometry_msgs/Pose`](https://docs.ros.org/en/api/geometry_msgs/html/msg/Pose.html)                                           | `map` | `1`   | <ul><li>`Reliable`</li><li>`Volatile`</li><li>`Keep last/1`</li> |
| `/sensing/gnss/pose_with_covariance` | [`geometry_msgs/PoseWithCovarianceStamped`](https://docs.ros.org/en/api/geometry_msgs/html/msg/PoseWithCovarianceStamped.html) | `map` | `1`   | <ul><li>`Reliable`</li><li>`Volatile`</li><li>`Keep last/1`</li> |
| `/sensing/gnss/orientation` | [`autoware_sensing_msgs/GnssInsOrientationStamped`](https://docs.ros.org/en/humble/p/autoware_sensing_msgs/msg/GnssInsOrientationStamped.html) | `map` | `0(1)`   | <ul><li>`Reliable`</li><li>`Volatile`</li><li>`Keep last/1`</li> |
