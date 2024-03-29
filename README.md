# The CogniFly Project
<img src="imgs/header.png" width="100%" style="margin: 20px auto 20px; display: block;" alt="CogniFly Logo"/>

The CogniFly project's purpose is ambitious: a unique platform for experimenting with autonomous flying robots. Using a bioinspired exoskeleton concept, our open source quadcopter combines lightweight with an unusual flexible frame. CogniFly is able to continue flying, even after a serious crash, thanks to its exceptional capacity of absorbing impact energy. Specially created to avoid the new limits for recreational RC aircraft heavier than 250 grams, our drone integrates a Raspberry Pi Zero W, Raspberry Pi Camera V2, and a Google’s AIY Vision Bonnet (powered by Intel Movidius), so it can perform computationally complex tasks such as image recognition *on-the-fly*. In addition to all that, CogniFly was specially designed allowing it to have its battery automatically swapped by a bespoke system.


## Why did we decide to create yet another drone design?
It all started when the then [MISTLab](http://mistlab.ca/) postdoctoral researcher, [Ricardo de Azambuja](https://github.com/ricardodeazambuja), was awarded with the prestigious [IVADO.ca (postdoctoral scholarship 2019/2020)](https://ivado.ca/en/ivado-scholarships/postdoctoral-scholarships/) for the project ["High Fidelity Data Collection for Precision Agriculture with Drone Swarms"](https://ricardodeazambuja.com/projects/razambuja_ivado2019/). That project was originally planning to employ *off-the-shelf* drones, more precisely the [DJI Tello](https://store.dji.com/product/tello) with the hope it would be possible to customize it and control a swarm using [Buzz](https://github.com/MISTLab/Buzz). However, as soon as the project started, it became clear we wanted a drone capable to be as autonomous as possible running Buzz internally, therefore, a drone that needed to be hooked to a laptop was not what we were aiming for. By the time the project started (March 2019), regulations in Canada would put harsh restrictions on drones above 250g. Moreover, we wanted an *AI* drone capable to run deep neural models (e.g. object detection) on-board because that was a main part of our initial proposal. Another reason to jump into the design of a totally new [UAV](https://en.wikipedia.org/wiki/Unmanned_aerial_vehicle) was our dream of having an open-source design, under 250g that could be easily repaired or customized with a battery holder that would allow the battery to be automatically swapped. 

<img src="imgs/CogniFlyField.png" width="80%" style="margin: 20px auto 20px; display: block;" alt="Precision Agriculture with CogniFly"/>

## The CogniFly
CogniFly is based on small, lightweight off-the-shelf flight controllers (FC) that became very popular (and cheap!) thanks to the FPV drone community (and some massive hardware cloning). A FC is mainly responsible to keep the drone stable by calculating motor speed commands based on the data received from sensors like the [IMU](https://en.wikipedia.org/wiki/Inertial_measurement_unit). Those commands, in the case of a [quadcopter](https://en.wikipedia.org/wiki/Quadcopter), go straight to the [ESC](https://en.wikipedia.org/wiki/Electronic_speed_control) that converts them into something that actually makes the [motors spin accordingly](https://en.wikipedia.org/wiki/Quadcopter#Flight_dynamics). Some FCs (actually just by changing the firmware) are capable of following certain paths (waypoints) if they have an external positioning sensor (e.g. a [GPS](https://en.wikipedia.org/wiki/Global_Positioning_System)). However, a FC alone can not do much more than that and an external pilot needs to take a lot of decisions and send commands to the FC typically from a joystick-like device that transmits the commands commonly through radio signals. The external pilot doesn't need to be a human being and, in fact, it doesn't need to send commands through radio signals either (it could be through a cable). CogniFly uses a single-board computer (SBC), in this case a [Raspberry Pi (RPI) Zero W](https://www.raspberrypi.org/products/raspberry-pi-zero-w/), that pretends to be the external pilot. 

The RPI sits on the top of the drone and it's wired to the FC through its serial port ([UART](https://en.wikipedia.org/wiki/Universal_asynchronous_receiver-transmitter), to be more precise). In addition to the RPI, we also use [a sensor](http://www.mateksys.com/?portfolio=3901-l0x) that provides altitude ([Time-of-Flight sensor](https://www.terabee.com/time-of-flight-principle/)) and relative odometry ([optical flow](https://en.wikipedia.org/wiki/Optical_flow) sensor) allowing the FC to keep the drone in a fixed position thus transforming the pilot's task into a high level one. Some extra nice advantages of using a RPI Zero W as the SBC is the [huge community](https://magpi.raspberrypi.org/articles/raspberry-pi-sales) around it and easy access to WiFi and Bluetooth out of the box together with SPI, I2C, UART, PWM, etc. In order to allow the RPI to control the drone we developed a special [Python3](https://www.python.org/doc/sunset-python-2/) library called [YAMSPy](https://github.com/thecognifly/YAMSPy) that *speaks* the [MultiWii](https://github.com/multiwii) protocol and can send commands and read information from a FC running [iNAV](https://github.com/iNavFlight/inav) or [Betaflight](https://github.com/betaflight/betaflight). 

By choosing to use a RPI as the SBC we openned up many possibilities for the AI side of this project because it's very easy to interface external hardware to the RPI like the [AI Vision Kit](https://aiyprojects.withgoogle.com/vision), [Coral EdgeTPU USB Accelerator](https://github.com/ricardodeazambuja/libedgetpu-rpi0/releases/tag/rpi0_tflite_edgetpu) ([or simply run TensorFlow Lite directly](https://www.tensorflow.org/lite/guide/build_rpi)) and there're already some examples available. After studying possible hardware accelerators for running inference on the edge that were small and lightweight enough (remember the 250g limit), we ended up with these three options: [JeVois](http://jevois.org/), [Google AIY Vision Kit](https://aiyprojects.withgoogle.com/vision/) and [Google Coral EdgeTPU USB](https://coral.ai/products/accelerator/). So far we have successfully tested Google's AIY Vision Kit because it interfaces very nicely with the RPI Zero W, [it gives us access to extra GPIO pins (with ADC and PWM)](https://aiyprojects.readthedocs.io/en/latest/vision.html#vision-bonnet) and even a [crypto chip](https://www.microchip.com/wwwproducts/en/ATECC608A), but we have plans to test the other options in the near future ([see this repo for EdgeTPU USB on the RPI Zero](https://github.com/ricardodeazambuja/libedgetpu-rpi0/releases/tag/rpi0_tflite_edgetpu)).

In addition to the electronic boards, battery, motors and propellers, a quadcopter needs a frame where all the parts are held together. Nowadays, the most common solution is a carbon fiber sheet that is machined to a certain shape (usually something resembling a 'X') with holes for connecting motors and the FC. However, our aim was to design a customizable frame that could be easily 3D printed by any small hobby grade machine (e.g. [Tinyboy Education Project](https://www.tinyboy.net/)). The reason for that was to allow us to modify or fix the drone easily and iterate the design faster since we already had a small printer available (an [MP Select Mini V2](https://www.monoprice.com/product?p_id=21711)). 

As the project progressed, we rapidly faced the fact that experimental drones crash a lot more than what we were used to when flying off-the-shelf ones. However, not only the frame needs to withstand daily (hourly...) crashes. An experimental design ideally requires some protection to propellers and other components to avoid breaking things each time an accident occurs. Additionally to protecting the more vulnerable parts from direct impacts, it's necessary to avoid abrupt (de)acceleration during impacts because that stress can cause all sorts of components to fail prematurely too.

Nature probably had these problems during its evolutional design process. Bees are a cool example of nature's design choices. YouTube is full of videos where [bees crash land](https://www.youtube.com/watch?v=A4mcsi_DHKA) or simply [hit each other while they are flying](https://youtu.be/mZhwK_y3a2Y?t=23) clearly without dying or breaking anything. We definitely are not the first ones to take notice of [bees funny flying behaviours](https://spectrum.ieee.org/automaton/robotics/drones/the-secret-to-small-drone-obstacle-avoidance-is-to-just-crash-into-stuff) and, to be fair, the whole idea of being somehow soft while crashing [seens to come from the 30's](https://en.wikipedia.org/wiki/Crumple_zone#Early_development_history). However, one stricking difference between what bees do and crumple zones is that bees come out intact from the incidents while crumple zones dissipate the energy by destroying themselves.

Our current design mixes soft and hard materials thus making CogniFly a drone capable of stunts that are quite unique while keeping its total weight just under 250g. Below is depicted the version used in the crash tests for our [ICRA 2022 paper](https://arxiv.org/abs/2103.04423):

<img src="imgs/CogniFly.jpg" width="80%" style="margin: 20px auto 20px; display: block;" alt="CogniFly"/>

And here is [CogniFly's latest iteration](https://github.com/thecognifly/CogniFly-STL/tree/master/Latest):
<a href="https://www.youtube.com/watch?v=jgervD5KhRY"><img src="imgs/latest.jpeg" width="80%" style="margin: 20px auto 20px; display: block;" alt="CogniFly"/></a>


## Collision Resilient
<img src="imgs/cognifly_small.gif" width="80%" style="margin: 20px auto 20px; display: block;" alt="CogniFly Table Crash"/>

## Raspberry Pi Zero W and AI *on-the-fly*
<img src="imgs/ai_on-the-fly.jpg" width="80%" style="margin: 20px auto 20px; display: block;" alt="AI On-The-Fly"/>

## Designed for Automatic Battery Swapping
<img src="imgs/swap_small.gif" width="80%" style="margin: 20px auto 20px; display: block;" alt="CogniFly Battery Swapping"/>

<img src="imgs/Battery_Sliding_Perspective.gif" width="80%" style="margin: 20px auto 20px; display: block;" alt="CogniFly Battery Slide"/>

## Videos!
### [ICRA 2022 Video Teaser](https://www.youtube.com/watch?v=jgervD5KhRY):
<a href="https://www.youtube.com/watch?v=jgervD5KhRY"><img src="imgs/table_crash.gif" width="80%" style="margin: 20px auto 20px; display: block;" alt="CogniFly Video Teaser"/></a>

### [ICRA 2022 Video Presentation](https://www.youtube.com/watch?v=HAg0lfTfMrk):
<a href="https://www.youtube.com/watch?v=HAg0lfTfMrk"><img src="imgs/ICRA 2022 Poster.jpg" width="80%" style="margin: 20px auto 20px; display: block;" alt="CogniFly Video Teaser"/></a>

### [Design evolution, or lessons learned](https://youtu.be/8DbwKIAxxqc)
We went through many iterations before we found the sweet spot. The slideshow below shows since the very first design until the one using 3D printed flexible nets: 
<a href="https://youtu.be/8DbwKIAxxqc"><img src="imgs/vlcsnap-2021-03-07-12h10m46s022.png" width="80%" style="margin: 20px auto 20px; display: block;" alt="Slideshow CogniFly Evolution"/></a>

## Useful links
- [The CogniFly Project](https://github.com/thecognifly/)
- [cognifly-python](https://github.com/thecognifly/cognifly-python)
- [YAMSPy](https://github.com/thecognifly/YAMSPy)
- [CogniFly-STL: How to print / assemble the drone](https://github.com/thecognifly/CogniFly-STL)
- [iNav Fork customized for CogniFly](https://github.com/thecognifly/inav/tree/CogniFly)
- [How to train your own object detector using the AIY Vision Kit](https://github.com/thecognifly/AIY-Vision-Kit-Utils)
- [Many Neural Network models available from Google AIY project](https://github.com/google/aiyprojects-raspbian/tree/aiyprojects/src/examples/vision)
- [How to use docker to speed up compilation for the RPI Zero](https://ricardodeazambuja.com/rpi/2020/12/29/rpi2docker/)
- [If you are curious about our logo, it's not a bee or wasp](https://en.wikipedia.org/wiki/Mallophora_bomboides)


## Publications
- [de Azambuja, R., Fouad, H., & Beltrame, G. (2021). When Being Soft Makes You Tough: A Collision Resilient Quadcopter Inspired by Arthropod Exoskeletons. arXiv preprint arXiv:2103.04423.](https://arxiv.org/abs/2103.04423)
- [H. Fouad and G. Beltrame, "Energy Autonomy for Robot Systems With Constrained Resources," in IEEE Transactions on Robotics, 2022, doi: 10.1109/TRO.2022.3175438.](https://ieeexplore.ieee.org/abstract/document/9813363)


## Acknowledgements
The CogniFly Project is built around many other succesful open-source projects starting by [Linux itself](https://www.linuxfoundation.org/). Therefore, it would be impossible to cite all projects here and we apologize beforehand because we are going, for sure, to forget to cite many: the flight controllers used in this project run [iNAV](https://github.com/iNavFlight/inav) or [Betaflight](https://github.com/betaflight/betaflight) (and those projects themselves only exist because there was a [MultiWii project](https://github.com/multiwii) before them). The RPI Zero W image was based on the one supplied by [Google AIY Projects](https://github.com/google/aiyprojects-raspbian). Many ideas for the code probably are from [stackoverflow](http://stackoverflow.com/) and inspiration to YAMSPy definitely came from my friend Aldo's [pyMultiWii](https://github.com/alduxvm/pyMultiWii). The [PiDrone project](https://github.com/h2r/pidrone_pkg) and [Pidrone (yup, same names!)](https://github.com/PiStuffing/Quadcopter) also gave us a lot of inspiration since they were using RPI 3 to control drones. Thanks to the wonderful [Raspiberry Pi Foundation](https://www.raspberrypi.org/) we have an affordable and small single-board computer (the Raspberry Pi Zero W). Nowadays 3D printers are only accessible to so many people because of the [RepRap project](https://reprap.org/wiki/RepRap).

Many people from [MISTLab](http://mistlab.ca/) helped during the development of this project, so a big thanks goes to the whole team. 

Logo design by [Daniele Sinhorelli](http://www.danielesinhorelli.me/booksite/).

This work was only possible thanks to the financial support from [IVADO.ca (postdoctoral scholarship 2019/2020)](https://ivado.ca/en/ivado-scholarships/postdoctoral-scholarships/).

---
## Media coverage (20/03/2021)
- <https://hackaday.com/2021/03/15/resilient-ai-drone-packs-it-all-in-under-250-grams/>
- <https://www.hackster.io/news/creating-autonomous-flying-robots-with-the-cognifly-project-b12a0a3f9c7d>
- <https://www.reddit.com/r/robotics/comments/m2yrbf/the_cognifly_project/>
- <https://10-raisons.fr/le-drone-ia-resilient-regroupe-tout-en-moins-de-250-grammes/>
- <https://blog.dronestagr.am/2021/03/15/resilient-ai-drone-packs-it-all-in-under-250-grams/>
- <https://intofpv.com/t-the-cognifly-project>
- <https://bartonspringstribune.com/resilient-ai-drone-packs-everything-in-under-250-grams/>
- <https://www.crackedconsole.com/2021/03/17/creating-autonomous-flying-robots-with-the-cognifly-project/>
- <https://www.facebook.com/mistlab.ca/videos/443400980279219/>
- <https://www.linkedin.com/posts/gbeltrame_ai-robotics-activity-6775847096106008577-yQln>
- <https://twitter.com/jumpjoe78/status/1370100314741833742>
