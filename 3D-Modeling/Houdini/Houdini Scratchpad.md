# Accessing Iterations in a node loop

To access the iterations of a loop you have to set up a few things.
![[attachments/Pasted image 20221021115511.png]] Select the *Block Begin* of the loop nodes. The click the *Create Meta Import Node* button in the properties window. 
![[attachments/Pasted image 20221021115551.png]]
This will create a *Meta Import Node* 
![[attachments/Pasted image 20221021115627.png]]
If we view node info for that node we will see two imortant *Detail Attributes*: *iteration* and *numiterations* which are both *integers*.
![[attachments/Pasted image 20221021115750.png]]
Because these are detail attributes we can access them throughout our network with a [detail](https://www.sidefx.com/docs/houdini/expressions/detail.html) expression function.
![[attachments/Pasted image 20221021122343.png]]
1. A *string* path pointing to the correct *metadata* node of the loop.
2. The *string* name of the detail attribute we want to grab. In this case *iteration*
3. The *attribute index*, in this case not applicable, but if it was a vector or something else we could use this here.


![[attachments/Pasted image 20221021122629.png]]
Now we can see the iterations in that parameter field.


---
# Curve Vectors
#procedural-modeling #houdini #curves
A common problem you may encounter is adding the correct vector data to a curve to get the *up* and *banking* directions per point.
There are many ways to do this, this is one way I've solved it.
Start with some sort of curve or line. ![[attachments/Pasted image 20221021175618.png]]
Once you have a curve drop down a [Resample](https://www.sidefx.com/docs/houdini/nodes/sop/resample.html) node. 
![[attachments/Pasted image 20221021180749.png]]
In the resample node there are two things we need to do.
1. Tread polygons as *Subdivision Curves*
2. Change the tangent attribute from *tangentu* to *N* (Normal)
![[attachments/Pasted image 20221021180949.png]]
Now that that is setup. Drop down an [Attribute Wrangle](https://www.sidefx.com/docs/houdini/nodes/sop/attribwrangle) node.
![[attachments/Pasted image 20221021183239.png]]
Make sure the wrangle is running over *points*.
![[attachments/Pasted image 20221021210701.png]]
```vex
vector flatnorm = @N;
flatnorm.y = 0;
flatnorm = normalize(flatnorm);

v@right = cross({0,1,0}, flatnorm);
v@up = cross(@N, @right);

v@oldnorm = @N;
v@N = v@right;
```

>We are saving the *oldnormal* to use it later

>[!NOTE] You could add this as a preset -> [[node-presets]]

This is some standard linear algebra. Here's what's going on.
We make a new vector variable that's the same as the *normals* on the points. Then set the *Y* part of the vector to 0, then [normalize](https://www.sidefx.com/docs/houdini/vex/functions/normalize.html) our vector. Then we create two new vector attributes. this is done with `v@something` syntax. The first is a [Cross product](https://en.wikipedia.org/wiki/Cross_product) of our first variable and a simple '{0,1,0}' vector. The second is a cross product of our *normal* and the last attribute we just made.
Now our normals look like this ->
![[attachments/Pasted image 20221021190724.png]]
![[attachments/Pasted image 20221021191325.png]]
Now that our points have the correct data we can start to use that data.
Drop down a [Convert Line](https://www.sidefx.com/docs/houdini/nodes/sop/convertline.html) SOP node.
![[attachments/Pasted image 20221021193055.png]]
Convert line will split the line into separate primitives.
Before the *convert line* node
![[attachments/Pasted image 20221021193307.png]]
After the *convert line* node
![[attachments/Pasted image 20221021193339.png]]
*Convert line* creates a *restlength* attribute.
![[attachments/Pasted image 20221021193442.png]]
This will be very helpful later.

In the wrangle let's add the following line at the end"
`v@N = v@right;`
![[attachments/Pasted image 20221021193857.png]]
Then drop down a [Peak](https://www.sidefx.com/docs/houdini/nodes/sop/peak.html) node. 
![[attachments/Pasted image 20221021193947.png]]
 Scale the distance value a bit.
 ![[attachments/Pasted image 20221021194246.png]]
 It should like this in the viewport:
 ![[attachments/Pasted image 20221021194459.png]]
![[attachments/Pasted image 20221021194734.png]]
Next drop down another *Attribute wrangle* node.
![[attachments/Pasted image 20221021194932.png]]
and attach both the original and the peaked stream.
Here we will be comparing the length of the two primitives.

>[!IMPORTANT] Make sure this wrangle is running over *primitives* ![[attachments/Pasted image 20221021200321.png]]


```vex
float otherlength = prim(1, "restlength", @primnum);
f@bankratio = f@restlength/otherlength;
```
![[attachments/Pasted image 20221021195336.png]]
The first line we are using the [prim](https://www.sidefx.com/docs/houdini/vex/attrib_suite#prim) function. 
![[attachments/Pasted image 20221021195526.png]]
The 1 indicates we are using the *second* input of the wrangle.
![[attachments/Pasted image 20221021195601.png]]
![[attachments/Pasted image 20221021195709.png]]
*restlength* is the attribute we are looking for. The one created from the *convert line* node.
![[attachments/Pasted image 20221021195759.png]]
`@primnum` means we want to do this for each *primitive*.
![[attachments/Pasted image 20221021195854.png]]
Next we need to drop down an [Attribute Promote](https://www.sidefx.com/docs/houdini/nodes/sop/attribpromote.html) SOP. 
![[attachments/Pasted image 20221021200801.png]]
And then make sure the we are promoting *bankratio* from *primitive* to *point*.

>[!NOTE]- creating a subnet
>If you wish you can put everything into a [Subnet](https://www.sidefx.com/docs/houdini/nodes/obj/subnet.html) by selecting all the nodes you want to use and pressing *shift+c*
>![[attachments/Pasted image 20221021201038.png]]
>This will make it easier manage and create a [digital asset](https://www.sidefx.com/docs/houdini/assets/intro.html) with it
>I've created a *subnet* for the *bank ratio* stuff:
![[attachments/Pasted image 20221021201433.png]]

Next drop down an [Attribute Transfer](https://www.sidefx.com/docs/houdini/nodes/sop/attribtransfer.html) SOP.
![[attachments/Pasted image 20221021201615.png]]
We want to transfer the *bankratio* attribute to the other curve.
![[attachments/Pasted image 20221021201717.png]]

>[!NOTE] Visualizers are an important part of a Houdini workflow. see [[houdini visualizers]] and [Visualizers documentation](https://www.sidefx.com/docs/houdini/basics/visualizers.html)

Let's drop down a [Visualizer](https://www.sidefx.com/docs/houdini/basics/visualizers.html) node.
![[attachments/Pasted image 20221021202042.png]]
![[attachments/Pasted image 20221021202129.png]]
This will let us see the values of our *attribute* in the viewport.
![[attachments/Pasted image 20221021202411.png]]
Next we need to remap these values into a usable range. *-1-1* that way math will work better down the line.
>[!NOTE] -1-1 and 0-1 range is also very common programming, gamedev, and CG because it gives you a good starting point for other mathematics.

>[!NOTE] Remapping Values
>
>there are many ways to remap values in houdini.
>see [[remapping values in houdini]]

Let's drop down an *attribute promote* SOP.
![[attachments/Pasted image 20221021202920.png]]
![[attachments/Pasted image 20221021203133.png]]
We'll take the *bankratio* attribute and promote it to a *detail* attribute. Then set the *promotion method* to *maximum*. This will get the highest value. Then create a new attribute called *maxratio* and make sure we don't delete the old one.
Then copy that node and change it so we get the *minimum*.
![[attachments/Pasted image 20221021203612.png]]
![[attachments/Pasted image 20221021203631.png]]
Now in our [Geometry Spreadsheet pane](https://www.sidefx.com/docs/houdini/ref/panes/geosheet.html) we can see our *detail* attributes.
![[attachments/Pasted image 20221021203751.png]]


Next drop down an [Attribute Remap](https://www.sidefx.com/docs/houdini/nodes/sop/attribremap.html) node.
![[attachments/Pasted image 20221021203402.png]]
Now we want to remap the max and min we just created.
![[attachments/Pasted image 20221021205905.png]]
We will remap *bankratio* to -1-1 and then use the *detail* function to get the detail attributes we made for min and max.
![[attachments/Pasted image 20221021205958.png]]
![[attachments/Pasted image 20221021210939.png]]
Now the *bankratio* is in usable values

![[attachments/Pasted image 20221021210058.png]]

Next we will drop down a wrangle to do the rotation.
![[attachments/Pasted image 20221021211502.png]]
![[attachments/Pasted image 20221021213202.png]]
```vex
@N = v@oldnorm;
matrix rot = ident(); //this creates a rest rotation. or no rotation
rotate(rot, f@bankratio * chf('BankAmount'), @N);
/*rotate the rot matrix by the bankratio using the normal as the axis */
v@right *= rot; //multiplying the vector by the rotation matrix we made
v@up *= rot;

```
We are basically rotating our vectors using the rotation matrix we created with the [rotate](https://www.sidefx.com/docs/houdini/vex/functions/rotate.html) vex function.
![[attachments/Pasted image 20221021211528.png]]
>[!NOTE] You may need to `f@bankratio` to negative
>![[attachments/Pasted image 20221021213450.png]]
>if it banks the wrong way

Then drop down a [Sweep](https://www.sidefx.com/docs/houdini/nodes/sop/sweep.html) node.
![[attachments/Pasted image 20221021213623.png]]
set it to *ribbon* to see the banking.
![[attachments/Pasted image 20221021213658.png]]
![[attachments/Pasted image 20221021213715.png]]
