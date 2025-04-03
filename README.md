# SVRaster WebGL Viewer

A WebGL-based viewer for visualizing sparse voxel scenes from the Nvidia [Sparse Voxels Rasterization paper](https://svraster.github.io/). This viewer provides an interactive way to explore and visualize the voxel radiance field from the web.. You can try the viewer at [vid2scene.com/voxel](https://vid2scene.com/voxel)

The rendering isn't exactly the same as the reference CUDA implementation, but its pretty similar. Also, only SH1 is supported.

## Features

- Interactive camera controls:
  - Left-click + drag: Orbit camera
  - Right-click + drag: Pan camera
  - Mouse wheel: Zoom
  - WASD/Arrow keys: Move camera
  - Q/E: Rotate scene around view direction
  - Space/Shift: Move up/down
- Touch controls for mobile devices:
  - 1 finger drag: Orbit
  - 2 finger drag: Pan/zoom
- Performance metrics display (FPS counter)


## Implementation and Performance Notes

- This viewer uses a depth-based sorting approach rather than the ray direction-dependent Morton ordering described in the paper
- The current implementation has only the most basic optimizations applied - there's significant room for performance improvements. Right now, the fragment shader is the bottleneck. Memory usage could also be lowered because nothing is quantized right now. If you have a perf improvement suggestion, please feel free to submit a PR! 
- It runs at 60 FPS on my laptop with a Laptop 3080 GPU
- It runs at about 15 to 20 FPS on my iPhone 13 Pro Max


You can pass ?samples=X as a URL param which will adjust the amount of density samples per ray in the fragment shader. The default is 3, but you can get a pretty good performance increase by decreasing this value, at the cost of a little less accurate rendering.

## URL Parameters

The viewer supports a few URL parameters to customize its behavior:

- `?samples=1` - Adjusts the amount of density samples per ray in the fragment shader (default: 3). Lower value increases performance, at a slight cost of rendering accuracy.
- `?url=https://example.com/myply.ply` - Loads a custom PLY file from the specified URL (default: pumpkin_600k.ply from HuggingFace here: https://huggingface.co/samuelm2/voxel-data/blob/main/pumpkin_600k.ply). For example, I also have a 1.2 million voxel pumpkin ply which you can load with this url: https://vid2scene.com/voxel/?url=https://huggingface.co/samuelm2/voxel-data/resolve/main/pumpkin_1200k.ply
- `?showLoadingUI=true` - Shows the PLY file upload UI, allowing users to load their own files

## How to Try Yourself

If you have your own SVRaster scenes that you'd like to visualize in this WebGL viewer, you can use this forked version of SVRaster that supports PLY export:

1. Clone the forked SVRaster repository: [github.com/samuelm2/svraster](https://github.com/samuelm2/svraster)
2. Run SVRaster with SH degree 1 to create a model compatible with this viewer:
   ```bash
   python train.py --source_path /PATH/TO/COLMAP/SFM/DIR --model_path outputs/pumpkin/ --sh_degree 1 --sh_degree_init 1 --subdivide_max_num 600000
   ```
   The PLY file will be automatically saved in your outputs model_path directory.

3. For existing SVRaster models that were previously trained, you can use the `convert_to_ply.py` script:
   ```bash
   python convert_to_ply.py outputs/pumpkin/ outputs/pumpkin/pumpkin.ply
   ```

4. Open the WebGL viewer and use the URL parameter or file upload UI (or modify the code itself) to load your custom PLY file

Note: The PLY files generated by this process are very unoptimized and uncompressed, so they can get very large quickly. I usually keep the number of voxels down to 600k to 1m range using the subdivide_max_num flag above.

## Other note

This project was made with heavy use of AI assistance ("vibe coded"). I wanted to see how it would go for something graphics related. My brief thoughts: it is super good for the boilerplate (defining/binding buffers, uniforms, etc). I was able to get simple rendering within hours. But when it comes to solving the harder graphics bugs, the benefits are a lot lower. There were multiple times where it would go in the complete wrong direction and I would have to rewrite portions manually. But overall, I think it is definitely a net positive for smaller projects like this one. In a more complex graphics engine / production environment, the benefits might be less clear for now. I'm interested in what others think.