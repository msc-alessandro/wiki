## Concerns

### **Retirado de: A Scalable Distributed Architecture Towards Unifying IoT Applications**
*  Heterogeneity
*  Scalability
*  Interoperability
*  Security & Privacy

### **Retirado de: A Scalable Framework for Provisioning Large-Scale IoT Deployments**

*  Atualização de software/deploy de novas funcionalidades

>  Currently, configuration and provisioning of IoT devices must largely be
performed manually, making it difficult to quickly react to changes in application or
infrastructure requirements. Moreover, we see the emergence of IoT devices (e.g., Intel
IoT gateway,3 SmartThings Hub,4 and Raspberry Pi5) that offer functionality beyond
basic connected sensors and provide constrained execution environments with limited
processing, storage, and memory resources to execute device firmware. These currently
unused execution environments can be incorporated in IoT systems to offload parts of
the business logic onto devices. In the context of our work, we refer to these devices as
IoT gateways.


### **Retirado de: A Simulation Platform for Large-Scale Internet of Things Scenarios in Urban Environments**


*  Quantidade de conexões


![Simulação](uploads/a43a3f3ab2ebf4d96042a634730917bf/Simulação.png)



>  The scenarios we have decided to simulate belong to a
typical application case of the Internet of Things in urban
environments: a smart parking infrastructure where in each
parking lot a sensor node is deployed to detect the presence
or absence of a vehicle [13]. Moreover, there are gateways
that take charge of collecting data from the sensor network,
and vehicles that move around the city and contact the gateways,
searching for an available parking lot.
In particular, we have defined that vehicle requests to
gateways are scheduled by an Homogeneous Poisson Process
with interarrival time being an exponentially distributed
random variable whose mean interarrival time is equal to
5 minutes. When a vehicle makes a request, it communicates
with the geographically nearest gateway. The number
of vehicles looking for a parking lot is chosen so that, at the
end of the simulation, 10% of the traveling vehicles have issued
a parking request to the gateways. Next, the gateway
contacts the parking sensors under its own responsibility and
waits for their response. Therafter, it communicates a response
to the vehicle. All this happens for a lifetime of the
simulation equals to 4 hours...

>  In order to assess the scalability of our simulation platform,
the number of nodes of the simulation has been varied,
as reported in Table 1, and the required computation
time has been measured. Simulations have been executed
on a server with 2 GHz Xeon CPU, 16 GB RAM, Ubuntu
GNU/Linux operating system.


### **Retirado de: Capillary Networks - Bridging the Cellular and loT Worlds**


> Cellular communication technologies can play a crucial
role in the development and expansion of loT. Cellular networks
can leverage their ubiquity, integrated security, network
management and advanced backhaul connectivity capabilities
into loT networks. In this regard, capillary networks [5][
7] aim to provide the capabilities of cellular networks to
constrained networks while enabling the connectivity between
wireless sensor networks and cellular networks. Hence, a
capillary network provides local connectivity to devices using
short-range radio access technologies while it connects to the
backhaul cellular network through a node called Capillary
Gateway (CGW)

>  In our system, the distributed cloud for loT devices includes
both data centers (DCs), where larger amounts of data
from different sources can be processed and where also management
functions typically reside, as well as local compute
infrastructure, in particular within capillary networks. The
latter makes it possible to process data locally, for example, to
aggregate or filter sensor data, which can reduce the amount
of data that is sent upstream towards data centers. Moreover,
it enables also low-latency sensor-actuator control loops.
In particular, we use containers, such as Docker [18], for
packaging, deployment (including updates and new features),
and execution of software in our cloud. In general, containers
have a low overhead, which, in turn, allows a high density
of instances in DCs. Moreover, containers can be executed in
more constrained environments without hardware virtualization
support, such as in local CGWs.



### **Retirado de: Challenges and Opportunities in Edge Computing**

> Research in micro operating systems or microkernels can
provide inroads to tackling challenges related to deployment
of applications on heterogeneous edge nodes. Given
that these nodes do not have substantial resources like
in a server, the general purpose computing environment
that is facilitated on the edge will need to exhaust fewer
resources. The benefits of quick deployment, reduced boot
up times and resource isolation are desirable [47]. There is
preliminary research suggesting that mobile containers that
multiplex device hardware across multiple virtual devices
can provide similar performance to native hardware [48].
Container technologies, such as Docker31 are maturing and
enable quick deployment of applications on heterogeneous
platforms. More research is required to adopt containers as
a suitable mechanism for deploying applications on edge
nodes


### **Retirado de: End-to-End Management of IoT Applications**


![savi-iot](uploads/4708aa3bff30b6f7e43732174ceea191/savi-iot.png)

> Aggregators (i.e., IoT gateways) are located outside of
the cloud and may be deployed on single board computers
(e.g., Raspberry PIs), mini computers, or smart devices
(e.g., smart phones, TVS or refrigerators) to provide management,
connectivity, and data preprocessing for sensors’ data. 
The main responsibility of IoT middleware is to facilitate such functionalities at aggregators.
A worker node of a cluster based data processing platform
may be deployed on aggregators (i.e., Cluster worker).
An example for this can be a worker node of a Kafka3
cluster that is responsible for in-place data cleaning and
aggregation. The Aggregator Autonomic Manager component
is a part of the autonomic management system
which is responsible for monitoring performance metrics
of aggregators and scale related microservices if needs
be. IoT Control Services are the control services used by
the autonomic manager to scale or reconfigure microservices.
IoT Gateway Microservices are the part of the IoT
application that implements application functionalities at
this layer. For example, in our sample implementation of
the IoT application, we deploy Kafka as microservices at
Aggregators.