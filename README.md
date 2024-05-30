# R2D2 Depth: Studying Depth Values in DROID

This codebase is built on top of a slightly out-dated version of the publicly released DROID repo [here]([here](https://github.com/AlexanderKhazatsky/DROID)). The purpose of this codebase is to provide a way to generate stereo depth values for frames in the DROID dataset, visualize them, and save them out in a format that makes it easy for ingestion in VIDAR.

## Running the TRI learned stereo model and visualizing results

1. Run ```bash pip install -e . ``` in the root directory.
2. Make sure to additionally install torch 2.0.1 (the TRI stereo depth model is only tested with this)
3. Make sure you have the ZED Python API installed (https://www.stereolabs.com/docs/app-development/python/install). Note that this requires installing the ZED SDK.
4. Mount the lustre FSX instance with the stereo model checkpoint and DROID data pre-downloaded (or access DROID data some other way and fix the paths)

```bash
    # Register Lustre Package Repository
    wget -O - https://fsx-lustre-client-repo-public-keys.s3.amazonaws.com/fsx-ubuntu-public-key.asc | gpg --dearmor | sudo tee /usr/share/keyrings/fsx-ubuntu-public-key.gpg >/dev/null
    sudo bash -c 'echo "deb [signed-by=/usr/share/keyrings/fsx-ubuntu-public-key.gpg] https://fsx-lustre-client-repo.s3.amazonaws.com/ubuntu focal main" > /etc/apt/sources.list.d/fsxlustreclientrepo.list && apt-get update'
    # Install Client for *current* Kernel
    sudo apt install -y lustre-client-modules-$(uname -r)
    # Create Mount Point
    sudo mkdir -p /mnt/fsx
    sudo chown -R ubuntu /mnt/fsx
    # Mount Lustre (make sure you get the correct address from AWS Console)
    sudo mount -t lustre -o noatime,flock fs-0ee5fb54e88f9dd00.fsx.us-east-1.amazonaws.com@tcp:/kxvmdbev /mnt/fsx
```
   
5. Run ```bash python scripts/post_processing/run_stereo_model.py```

You should see an image saved as depth_image_grid.png comparing the ZED stereo model and TRI stereo model outputs on a random timestep for the trajectory corresponding to the DATA_PATH [here]([url](https://github.com/ashwin-balakrishna96/r2d2-depth/blob/main/scripts/post_processing/run_stereo_model.py#L17)).

## Generating Stereo Depth Data for Training in VIDAR

1. Do all of the steps in the previous section and then run ```bash python scripts/post_processing/get_rgbd_train_data.py```
2. Reformatted DROID data with depth images saved will be stored in the path corresponding to SAVE_PATH [here]([url](https://github.com/ashwin-balakrishna96/r2d2-depth/blob/main/scripts/post_processing/get_rgbd_train_data.py#L26)). Currently, this saves depth images and other associated information for 32 timesteps per trajectory due to memory constraints but that can be easily modified.
3. To load the data from the above format in a convenient way, see the dataloader [here]([url](https://github.com/TRI-ML/datasets/blob/kyle-ashwin/png_r2d2_dataloader/datasets/dataloaders/R2D2Dataset.py)).
