---
tags: [houdini, vfx, simulation, pop]
title: "Houdini Squib Setup"
---

Squibs are used in VFX for small explosions like gun shots and things. In practical effects squibs are small explosives that are rigged to blow at the right time. In VFX we often have to recreate these.


---
# Geo setup
![[attachments/Pasted image 20221018183521.png]]
1. Initial Geo -> In this case we are using a sphere and clip node to only get the top part of the sphere. This is because squibs typically explode from the ground.
	- ![[attachments/Pasted image 20221018183701.png]] -> We have turned the scale of the sphere down so that it makes sense in the real world.
	- ![[attachments/Pasted image 20221018183829.png]] -> The [Clip](https://www.sidefx.com/docs/houdini/nodes/sop/clip)node is used to cut the sphere and lower it to the ground plane.
2. In this section we are setting up some variation for the particle spawning down stream in the network.
		- ![[attachments/Pasted image 20221018184431.png]] -> the [Attribute VOP](https://www.sidefx.com/docs/houdini/nodes/sop/attribvop.html) node creates noise for the downstream operation.
		- ![[attachments/Pasted image 20221018184838.png]] -> This is a simple VOP setup. We are taking point position ([Geometry attributes](https://www.sidefx.com/docs/houdini/model/attributes.html)) and feeding it into a noise node (Which all of the parameters are promoted to the top of the VOP net). After that we feed into an [Absolute](https://www.sidefx.com/docs/houdini/nodes/vop/abs.html) node, which takes any negative value and makes it positive. After that we feed that into Color output which in Houdini is *Cd*.
			- In order to quickly promote all parameters in a VOP node right-click the node then do the following -> ![[attachments/houdini-promote-all-parms-vops.png]]
3. In this section we set up the Vectors we need for the Pop solver.
	- ![[attachments/Pasted image 20221018212030.png]] -> In the [Attribute Wrangle](https://www.sidefx.com/docs/houdini/nodes/sop/attribwrangle.html) make sure it's set to run over *points* and in the code window make sure you type the following
		```vex
		@v=@N;
		@v *= ch('scale');
		```
 
	- We will tune the scatter values later
	- In the [Attribute Delete](https://www.sidefx.com/docs/houdini/nodes/sop/attribdelete.html) node made sure in the points section you have *Cd* for color written. ![[attachments/Pasted image 20221018212934.png]]
4. Make sure you feed the stream into some sort of DOP network. This could be a plain DOP net or a configured POP network. The next section goes over the Pop solver setup.
---
# Pop solver setup

The POP network is technically in a [DOP Network](https://www.sidefx.com/docs/houdini/nodes/sop/dopnet.html) node. This type of node can contain any type of simulation system. For the squib, we will build a POP system in the DOPnet. 

>[!NOTE] A POP system is a simple particle simulation setup.


![[attachments/Pasted image 20221018183244.png]]
1. Pop Source
2. Pop Object
3. Pop Solver
4. Ground Plane
5. Gravity

