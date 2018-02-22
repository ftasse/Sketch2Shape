# Why?

<!-- You can use the [editor on GitHub](https://github.com/ftasse/Sketch2Shape/edit/master/README.md) to maintain and preview the content for your website in Markdown files.

Whenever you commit to this repository, GitHub Pages will run [Jekyll](https://jekyllrb.com/) to rebuild the pages in your site, from the content in your Markdown files.-->

As an aspiring artist, I have many times tried to find or create 3D content for the stories I wanted to tell. This starts with having the right 3D models. However you quickly find out that you have only two options: spend hours searching for the perfect model on sites like [Sketchfab](http://sketchfab.com) or  spend weeks/months learning a 3D Graphics tool like [Blender](http://blender.org).  Neither is particular appealing.

Any concept I have starts with a sketch. Hand-drawing is one of the easiest ways of communicating ideas. So what if I could turn my hand-drawn doodles to a 3D model? After all we humans can infer shapes from sketches. Now, as a research scientist, this is a problem I can investigate.

![alt Some sketches from the Quick draw dataset](https://ftasse.github.io/Sketch2Shape/quickdrawpreview.jpg  "Some sketches from the Quick draw dataset")

![alt Some 3D models from the Shapenet dataset](https://ftasse.github.io/Sketch2Shape/shapenetcore.png "Some 3D models from the Shapenet dataset")

# What?

The Sketch2Shape looks into how 3D models can be generated from hand-drawn sketches.  As a research scientist working at the intersection of Graphics, Vision & Machine Learning, I use the latest techniques in these fields.  The main objective is an AI that outputs a 3D volumetric mesh, given as input an image of  a hand-drawn sketch.  For this to happend we need:

* Part 1 -  A training dataset consisting for pairs of sketches and meshes.
* Part 2 -  A generative neural network model that will produce the right mesh given a sketch


# Who?

Just me, Flora Tasse. I hold a PhD in Computer Graphics/Vision from the University of Cambridge where I worked on an AI agent, Shape2Vec that retrieves the right 3D model from a database given queries such sketches, photos, depth maps or other 3D models.  Check out this [SIGGRAPH Asia '16 paper](http://www.cl.cam.ac.uk/research/rainbow/projects/shape2vec/)if you want to learn out we use semantic information to achieve that.

Now, I lead the technological endeavors of [Selerio](http://selerio.io) where we reconstruct 3D contextual scenes from videos for more immersive and magical AR experiences.

As for where I come from, Cameroon is my motherland and where my dream for merging the real and the virtual started.

#  Part 1-  A Sketch-to-Shape dataset

Because generating dataset is time-consuming and resources-draining, I decided to build on top of existing datasets. There is no dataset of pairs of sketches  and meshes. But we can use edge detection and Graphics rendering techniques to create computer-generated sketches from 3D meshes.  

However computer-generated sketches are quite different from hand-drawn sketches. The latter are more complex, less structured and can vastly vary from one user to another based on style and skills.  So we need to link our computer-generated sketches to hand-drawn sketches. Luckily image similarity is a well-understood problem in Vision and we will use deep features generated from a Convolutional neural network to match a dataset of hand-drawn sketches to our own computer-generated sketches from a mesh dataset.

We will use these two existing datasets:

* [Shapenet Core](https://www.shapenet.org/) a dataset of 48,600 single clean 3D models covering 55 common object categories, from Stanford University. In particular we will use a version of this dataset where models are represented by 256^3 voxels. This version is obtained from the [ICCV'17 Shape Reconstruction Challenge](https://shapenet.cs.stanford.edu/iccv17/)

* [Quick Draw](https://quickdraw.withgoogle.com/data): a dataset of 50 million drawings from the Google Quick Draw project. 

## Computer-generated sketches

Here, we are generating sketches, accross multiple view directions, from the 3D models dataset.  There are two approaches one can take to solve this:

###  3D Line rendering techniques

Line rendering is a well-understood method in 3D Graphics. Given a 3D mesh and a camera pose, we want to render an image that captures the silhouettes, ridges and contours of the mesh.  My open-source project [shape2d-tools](https://bitbucket.org/ftasse/shape2d_tools/src) does just that with two diffent method:

1- A simple silhouette rendering `Silhouette Window` that uses a specific geometry shader to render silhouette edges from a 3D geometry, based on the light direction and the vertex normals. Below is an example of sketches generated using this technique:

![alt A 3D mesh](https://ftasse.github.io/Sketch2Shape/images/silhouette_edges/M000019.png  "a 3D Mesh") ![alt Line rendering from position 1](https://ftasse.github.io/Sketch2Shape/images/silhouette_edges/M000019_1.png  "Line rendering from position 1") ![alt Line rendering from position 2](https://ftasse.github.io/Sketch2Shape/images/silhouette_edges/M000019_4.png  "Line rendering from position 2")

2- A more complex method, `Suggestive Contours` is based on [Suggestive Contours](http://gfx.cs.princeton.edu/proj/sugcon/) from Princeton. Beyond the usual silhouette edges, they detect and draw apparent ridges they compute from the geometry. This produces those suggestive contours a human artist typically have in their sketches.

Both techniques will be valid to genrate sketches from any 3D model. However, neither take into account texture information. The Shapenet Core models are textured, and these textures can infer more cues about the 3D shape.

###  2D Edge Detection

Rather than the above complicated rendering techniques, we simply render the 3D textured meshes into multiple images from different views, and then generate edges for each image using the very popular [Canny Edge Detector](https://docs.opencv.org/3.3.1/da/d22/tutorial_py_canny.html).  This is what it looks like:

![alt A rendering of a boat from position 1 ](https://ftasse.github.io/Sketch2Shape/images/silhouette_edges/M000102_010.jpg  "A rendering of a boat from position 1") ![alt Extracted edges](https://ftasse.github.io/Sketch2Shape/images/silhouette_edges/M000102_010_edges.jpg  "Extracted edges")  ![alt A rendering of a boat from position 2 ](https://ftasse.github.io/Sketch2Shape/images/silhouette_edges/M000102_001.jpg  "A rendering of a boat from position 2") ![alt Extracted edges](https://ftasse.github.io/Sketch2Shape/images/silhouette_edges/M000102_001_edges.jpg "Extracted edges")

In summary, from the 3D meshes dataset we obtain voxelized models (a 3D array of boolean values)   and  12 computer-generated sketches for each mesh.


![alt A textured airplane model ](https://ftasse.github.io/Sketch2Shape/images/silhouette_edges/M000003_001.jpg "A textured airplane model ") ![alt Voxels](https://ftasse.github.io/Sketch2Shape/images/silhouette_edges/M000003_voxels.jpg "Voxels") ![alt Extracted edges 1](https://ftasse.github.io/Sketch2Shape/images/silhouette_edges/M000003_001_edges.jpg  "Extracted edges 1") 

## Computer-generated to Hand-drawn 

In this section, we match each sketch in the hand drawings dataset to a computer-generated sketch, so that we can link hand drawings to volumetric meshes. This one-to-one correspondence will be need to train our AI with pairs of positive results.

To deal with similarity, we take advantage of convolutional neural network. We compute a descriptor for each computer-generated 
sketch by passing it through the vgg16 neural net, and retaining the output of the last fully connected layer fc6. This produces a 2096D vector for each computer-generated image. 

With the above CNN-based descriptors, we can now match a hand-drawing to the most similar computer-generated sketch. Similarity between CNN_descriptors is the Cosine Distance.  Here is how results looks like:

![alt Voxels](https://ftasse.github.io/Sketch2Shape/images/silhouette_edges/M000003_voxels.jpg "Voxels") ![alt Extracted edges 1](https://ftasse.github.io/Sketch2Shape/images/silhouette_edges/M000003_001_edges.jpg  "Extracted edges 1")   ![alt Similar hand-drawn sketch 1](https://ftasse.github.io/Sketch2Shape/images/11.png "Similar hand-drawn sketch 1") 

## Where can I get the Sketch2Shape dataset?

Coming soon.

#  Part 2-  A Sketch-to-Shape Generative Model

Coming soon

<div id="disqus_thread"></div>
<script>
    /**
     *  RECOMMENDED CONFIGURATION VARIABLES: EDIT AND UNCOMMENT THE SECTION BELOW TO INSERT DYNAMIC VALUES FROM YOUR PLATFORM OR CMS.
     *  LEARN WHY DEFINING THESE VARIABLES IS IMPORTANT: https://disqus.com/admin/universalcode/#configuration-variables
     */
   
    var disqus_config = function () {
        this.page.url = "https://ftasse.github.io/Sketch2Shape/";  // Replace PAGE_URL with your page's canonical URL variable
        this.page.identifier = "sketch2shape"; // Replace PAGE_IDENTIFIER with your page's unique identifier variable
    };
    
    (function() {  // REQUIRED CONFIGURATION VARIABLE: EDIT THE SHORTNAME BELOW
        var d = document, s = d.createElement('script');
        
        s.src = '//sketch2shape.disqus.com/embed.js';  // IMPORTANT: Replace EXAMPLE with your forum shortname!
        
        s.setAttribute('data-timestamp', +new Date());
        (d.head || d.body).appendChild(s);
    })();
</script>


<!--### Markdown

Markdown is a lightweight and easy-to-use syntax for styling your writing. It includes conventions for

```markdown
Syntax highlighted code block

# Header 1
## Header 2
### Header 3

- Bulleted
- List

1. Numbered
2. List

**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src)
```

For more details see [GitHub Flavored Markdown](https://guides.github.com/features/mastering-markdown/).

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/ftasse/Sketch2Shape/settings). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://help.github.com/categories/github-pages-basics/) or [contact support](https://github.com/contact) and we’ll help you sort it out.-->
<noscript>Please enable JavaScript to view the <a href="https://disqus.com/?ref_noscript" rel="nofollow">comments powered by Disqus.</a></noscript>
