Title: Process Pools
Tags: python

I read this beautiful article by Adam Geitgey about process pools. You can read the full article on his page. However, I wanted to have a quick note here as a good reminder to self.

**Note: Using a Process Pool requires passing data back and forth between separate Python processes. If the data you are working with can’t be efficiently passed between processes, this won’t work.**

####Regular method to create thumbnails in Python2.7

```python
import glob
import os
from PIL import Image

def make_image_thumbnail(filename):
 # The thumbnail will be named "<original_filename>_thumbnail.jpg"
 base_filename, file_extension = os.path.splitext(filename)
 thumbnail_filename = f"{base_filename}_thumbnail{file_extension}"
 # Create and save thumbnail image
 image = Image.open(filename)
 image.thumbnail(size=(128, 128))
 image.save(thumbnail_filename, "JPEG")
 return thumbnail_filename

# Loop through all jpeg files in the folder and make a thumbnail for each

for image_file in glob.glob("*.jpg"):
 thumbnail_file = make_image_thumbnail(image_file)
 print(f"A thumbnail for {image_file} was saved as {thumbnail_file}")
```


Time it to get a good measurement of total time taken. Keep in mind here that the above code still runs as a single process on a single CPU. So most of the compute power is idle.

```shell
$ time python thumbnails_1.py
```

####Creating thumbnails using Process Pools

```python
import glob
import os
from PIL import Image
import concurrent.futures

def make_image_thumbnail(filename):
 # The thumbnail will be named "<original_filename>_thumbnail.jpg"
 base_filename, file_extension = os.path.splitext(filename)
 thumbnail_filename = f"{base_filename}_thumbnail{file_extension}"
 # Create and save thumbnail image
 image = Image.open(filename)
 image.thumbnail(size=(128, 128))
 image.save(thumbnail_filename, "JPEG")
 return thumbnail_filename

# Create a pool of processes. By default, one is created for each CPU in your machine.

with concurrent.futures.ProcessPoolExecutor() as executor:
 # Get a list of files to process
 image_files = glob.glob("*.jpg")
 # Process the list of files, but split the work across the process pool to use all CPUs!
 for image_file, thumbnail_file in zip(image_files, executor.map(make_image_thumbnail, image_files)):
 print(f"A thumbnail for {image_file} was saved as {thumbnail_file}")
```

Always timeit.

```
$ time python thumbnails_2.py
```


####Where is this approach good?

* Grabbing statistics out of a collection of separate web server log files
* Parsing data out of a bunch of XML, CSV or json files
* Pre-processing lots of images to create a machine learning data set