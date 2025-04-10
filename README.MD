# DVS Gesture Dataset Implementation/Experiment using snntorch.spikevision
## Current Project progress
* Successfully loaded the Gesture dataset from the snntorch.spikevision 

------------------------------------
## DVS Gesture Dataset Loading Issues and Troubleshooting

When running the code to load the data from the dvsgesture library, no issues were encountered, where the following code snippets downloaded the dvsgesture.gz files and extrated it and proceded to create a 'dvsgesture.hdf5' file. But, a problem was encountered later when tried to access the data read from the file. 

```python
from snntorch.spikevision import spikedata

train_ds = spikedata.DVSGesture("data/dvsgesture", train=True,num_steps=500, dt=1000)
test_ds = spikedata.DVSGesture("data/dvsgesture", train=False,num_steps=1800, dt=1000)
```
------------------------------------
### Initial Problem:
When the following code was run to access the data for visualization purposes: 
```python
# Visualize the data
n = 1
#indexing the single sample
a = (train_data.dataset[n][0][:,0] + train_data.dataset[n][0][:,1])
```
THE ERROR: 
`KeyError: "Unable to synchronously open object (object 'labels' doesn't exist)"`
was encountered. 

------------------------------------
### Potential Solutions/trials leading up to solution
#### First idea:
Maybe the dataset didnt get installed properly or there was some sort of corruption in the h5df file creation, and hence I installed it directly from the IBM Box link for the gesture dataset: https://ibm.ent.box.com/s/3hiq58ww1pbbjrinh367ykfdf60xsfm8/folder/50167556794 

Yet, the error persisted. 

#### Next Trial: 
Using the h5py library, I accessed and read through the 'dvsgesture.hdf5' file, which again showed that there was no key/object with the name 'labels' in it. 

From these two tests, the error is not in the dataset, or in the file itself, it was in the creation of the h5df file. 

#### Next Trial: 
This was the last option as I didn't think this was possible. As it was suspected taht there was an error in the h5df file creation and the file is created in the first line of the code when you call the "spikedata.DVSGesture()" It was clear that the **changes needed to be made in the source code of the SNNTorch library inself.**

For which I reinstalled the snntorch library but the error persisted so I had to read through the code and figure it out.

------------------------------------

### FINAL SOLUTION
**This part took the longest time as I had to read through the spikevision source codes understand it and then make the necessary changes so I may be able to access the code for it**

And After reading and understanding the source codes, it was found that the error was in the dvsgesture.py file under the function create_event_hdf5(). Where the part that created the labels and other attributes for the file were commented off as there was an error that persisited with another function in the events_timeslices.py file within the same package. 

Hence, following are the screenshots of the files that were changed, and the packages that needed to be called correctly, for the proper access of the Gesture dataset using snntorch.spikevision. 

#### File Location 
One thing to note I am using an environment through anaconda, so you will have to find the site-packages of your environment(eg. pip)

dvs_gesture.py  ; containing the create_event_hdf5() function

```anaconda3\envs\DSVgesture\Lib\site-packages\snntorch\spikevision\spikedata\dvs_gesture.py```

event_timeslices.py  ; there are no particular changes made here, but is important to understand for the import in the dvs_gesture.py file

```C:\Users\jovin\anaconda3\envs\DSVgesture\Lib\site-packages\snntorch\spikevision\event_timeslices.py```

#### Correct import 
This was the initial problem where they were importing a function called get_tmad_slice which was not being used, hence change the line 20 from: 
![Line20: from ..events_timeslices import get_tmad_slices ](code_images\image-1.png)

To import the getslice from the event_timeslices file as shown here
![Line20: from ..events_timeslices import get_tmad_slices, get_slices](code_images\image.png)

#### Uncommenting the file creation parts
*Uncommenting lines 334 to 340*
<figure style="display: inline-block; margin-right: 10px;">
  <img src="code_images\image-2.png" alt="alt text" style="max-width: 300px; height: auto;">
</figure>
<figure style="display: inline-block;">
  <img src="code_images\image-3.png" alt="alt text" style="max-width: 300px; height: auto;">
</figure>


*Uncommenting lines 347 to 349*\
**Here, uncommenting line 350 will give you an error causing the the program to crash hence, do not uncomment line 350**
<figure style="display: inline-block; margin-right: 10px;">
  <img src="code_images\image-4.png" alt="alt text" style="max-width: 300px; height: auto;">
</figure>
<figure style="display: inline-block;">
  <img src="code_images\image-5.png" alt="alt text" style="max-width: 300px; height: auto;">
</figure>

*Uncommenting lines 366 to 374*
<figure style="display: inline-block; margin-right: 10px;">
  <img src="code_images\image-6.png" alt="alt text" style="max-width: 300px; height: auto;">
</figure>
<figure style="display: inline-block;">
  <img src="code_images\image-7.png" alt="alt text" style="max-width: 300px; height: auto;">
</figure>

------------------------------------

### ENDING NOTES
After all that, once you delete the current hdf5 file and rerun the initial lines of code calling DVSGesture from spikevision it should download the hdf5 files accurately and you should get the output as seen on the file dvs_animation.gif. 

------------------------------------