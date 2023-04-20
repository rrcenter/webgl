# pixijs

## batch-rendering-system

https://medium.com/swlh/inside-pixijs-batch-rendering-system-fad1b466c420

Batch rendering is what makes PixiJS the fastest WebGL drawing engine. It works by drawing groups of objects using their merged geometry. This technique reduces the number of draw calls and uploads to the GPU. There are a few limitations that come with batch rendering:

1. Batched objects must have similar geometries, the same shaders, and the same number of textures.
2. To maintain the order in which objects are being rendered, batched objects must be flushed prematurely if a dissimilar object is encountered.
3. Geometries must be merged into one large buffer.

### Batch rendering algorithm
As defined by ObjectRenderer , flush() is supposed to do the actual drawing. The whole rendering algorithm can be explained by understanding what occurs inside the two curly braces of this method. I’ve summarized it below:

1. Allocate an attribute buffer and index buffer that can store atleast all the attributes and indices in the current batch. This is done via the getAttributeBuffer and getIndexBuffer methods, which cache those buffers to prevent recurrent large allocations.
2. Loop though the array of objects to be rendered and form homogeneous groups of objects. By homogeneous, I mean objects that require the same WebGL state, e.g. the same blend mode and all. These groups may also be limited in size. These groups are stored as BatchDrawCall objects, and each group is rendered one-by-one (since a huge batch may not fit into the GPU, limited by its texture unit count).
3. While looping though each object, their geometry is also appended to one interleaved geometry (this.geometryClass), which will be uploaded once.
4. Loop through each group, upload the textures in the group, set the current WebGL state as required by it, bind the vertex/fragment shaders, and draw all the objects in the group using gl.drawElements . This function takes a start index and an index count, which allows us to render only one group at a time.
5. Reset the state.