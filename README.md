# nanopi-m4-ubuntu-base-minimal

Ubuntu 18.04 base minimal image for RK3399 (NanoPi M4 / NEO4)


OS Image for development with the following tidbits:

* Kernel 4.4.y (with the best of both world, rockchip bsp kernel + kernel.org 4.4.y)
* Gbps
* Wifi
* Bluetooth
* 3D mali gpu (fbdev / lxde)
* VPU (X11 / lxde)
* camera (WiP)

You need *wget* and *curl* installed to grab the files in a Linux distro.

Get the latest files by running (or seee below to fetch specific Release version files):


	wget $(curl -s https://api.github.com/repos/avafinger/nanopi-m4-ubuntu-base-minimal/releases/latest | grep -oP '"browser_download_url": "\K(.*)(?=")')


then

	sudo ./flash_sd.sh /dev/sdX (or /dev/mmcblkY) where X is a letter from b,c.. and Y is a number from 0,1..)



**WARNING**
Find the correct device letter for your USB SD CARD or you may wipe your hdd.


# Release v1.0

These are the raw files for building the OS Image with kernel 4.4.166-rockchip

# Release v1.1

* minor fixes and optimizations (4K, journaling , heart-beat)
* DHCP enabled (default)
* We're ready to roll


		user: ubuntu
		password: ubuntu


Get v1.1 files:

		wget $(curl -s https://api.github.com/repos/avafinger/nanopi-m4-ubuntu-base-minimal/releases | grep -oP '"browser_download_url": "\K(.*)(?=")' | grep v1.1)


Boot takes about 10 seconds or less depending on SD card in use. 
**Tip**: If you can't see anything on screen, type enter twice!


On first login:

	sudo apt-get update
	sudo apt-get dist-upgrade



**Tip**: You may need to update the file **resolver.conf** to reflect you network (DNS)



type in the shell:	df -lh


	Filesystem      Size  Used Avail Use% Mounted on
	/dev/mmcblk0p2  7.3G  582M  6.4G   9% /
	devtmpfs        963M     0  963M   0% /dev
	tmpfs           964M     0  964M   0% /dev/shm
	tmpfs           964M   17M  947M   2% /run
	tmpfs           5.0M     0  5.0M   0% /run/lock
	tmpfs           964M     0  964M   0% /sys/fs/cgroup
	/dev/mmcblk0p1  113M   21M   84M  20% /boot
	tmpfs           193M     0  193M   0% /run/user/1000



type in the shell: free -m


	              total        used        free      shared  buff/cache   available
	Mem:           1926          47        1798          24          80        1793
	Swap:             0           0           0



Now we can start installing and/or configuring things:

* ssh
* wifi
* 3D mali user space for OpenGLES 3.1 / OpenCL 1.2 ( fbdev and X11 )	
* bluetooth
* VPU ( encode / decoding )
* camera ( when ready !?! )

**Instructions and DEB packages coming soon**

# Remote access the board

Install ssh to be able to login remotely:


		sudo apt-get install ssh


from your PC:

		ssh ubuntu@IP
		where IP is the board IP assigned by DHCP


# 3D Mali (fbdev)

To be able to use OpenGL ES 2 / 3 we need to install the Mali user space lib.

* Download files:

		wget $(curl -s https://api.github.com/repos/avafinger/nanopi-m4-ubuntu-base-minimal/releases | grep -oP '"browser_download_url": "\K(.*)(?=")' | grep v1.2)

* Install the necessary packages:

		sudo apt-get install libjpeg-turbo8 libjpeg8 libpng16-16 libegl1 libegl-mesa0 libpng-dev libjpeg-dev

* Install Mali


		sudo dpkg -i --force-all mali-t86x-rk3399-linux-4.4.y_1.0-1.deb

  Ignore the warnings.


* Install enhanced htop:

  You can monitor the health of you RK3399 (CPU Temp, CPU Freq and CPU VCore, enter F2 and add these monitors)

		sudo dpkg -i htop_2.1.1-3_arm64.deb 


* Install tools for benchmarking mali on fbdev:


		sudo dpkg -i glmark2-data_2014.03+git20150611.fa71af2d-0ubuntu5_all.deb 
		sudo dpkg -i glmark2-es2-fbdev_2014.03+git20150611.fa71af2d-0ubuntu5_arm64.deb 


  If everithing is Okay, then you can test it:

		
		sudo glmark2-es2-fbdev (test it remotely)

# Wifi

  Install packages:

		sudo apt-get install crda wpasupplicant


  Enable wifi editing the file /etc/network/interfaces and adding:


		allow-hotplug wlan0
		iface wlan0 inet dhcp
			wpa-ssid SID
			#psk="00012345678901234567890"
			wpa-psk afde07531d767050796db24c3d449e26a449c15b98bfaf20148970ec43242078
			dns-nameservers 8.8.8.8 8.8.4.4
			wireless-power off


  Where SID os you AP name and xxxxxxxxx is your encrypted password.
  Generate encrypted password like so:

		wpa_passphrase SSID xxxxxxxxx

  and finally boot and connect to Wifi

# GBM

  We have now fbdev working and with OpenGLES 3.2 (fbdev) we can go further and install GBM (Graphic Buffer Management) libraries
  to be able to use DRM for a non-Desktop environment (no X11/Wayland). VPU has support for DRM/GBM and not for fbdev yet.
   

  There is a DEB package ready to install VPU support. Mali user space for GBM is needed in order to be able to use GL and/or GLES 2/3.
  The nice thing here is that we can have Kodi Media Center to run on NanoPi M4 / NEO / T4 or any other RK3399 if you wish.

  Install the gbm libraries and VPU deb package so we can have Hardware Decoding and HW accelerated display with Mali T86x.


		sudo apt-get install libgbm-dev



  Test if GBM is fully working with **kmscube**

		./kmscube 
		Using display 0x55b89fca00 with EGL version 1.4
		===================================
		EGL information:
		  version: "1.4 Midgard-"r14p0-01rel0""
		  vendor: "ARM"
		  client extensions: "EGL_EXT_client_extensions EGL_EXT_platform_base EGL_KHR_client_get_all_proc_addresses EGL_KHR_platform_gbm EGL_MESA_platform_gbm"
		  display extensions: "EGL_KHR_partial_update EGL_KHR_image_pixmap EGL_EXT_image_dma_buf_import EGL_KHR_config_attribs EGL_KHR_image EGL_KHR_image_base EGL_KHR_fence_sync EGL_KHR_wait_sync EGL_KHR_gl_colorspace EGL_KHR_get_all_proc_addresses EGL_IMG_context_priority EGL_ARM_pixmap_multisample_discard EGL_KHR_gl_texture_2D_image EGL_KHR_gl_renderbuffer_image EGL_KHR_create_context EGL_KHR_surfaceless_context EGL_KHR_gl_texture_cubemap_image EGL_EXT_create_context_robustness EGL_KHR_cl_event2"
		===================================
		OpenGL ES 2.x information:
		  version: "OpenGL ES 3.2 v1.r14p0-01rel0-git(966ed26).2e3fa7564ebca70897f04bd9fb7bc67e"
		  shading language version: "OpenGL ES GLSL ES 3.20"
		  vendor: "ARM"
		  renderer: "Mali-T860"
		  extensions: "GL_ARM_rgba8 GL_ARM_mali_shader_binary GL_OES_depth24 GL_OES_depth_texture GL_OES_depth_texture_cube_map GL_OES_packed_depth_stencil GL_OES_rgb8_rgba8 GL_EXT_read_format_bgra GL_OES_compressed_paletted_texture GL_OES_compressed_ETC1_RGB8_texture GL_OES_standard_derivatives GL_OES_EGL_image GL_OES_EGL_image_external GL_OES_EGL_image_external_essl3 GL_OES_EGL_sync GL_OES_texture_npot GL_OES_vertex_half_float GL_OES_required_internalformat GL_OES_vertex_array_object GL_OES_mapbuffer GL_EXT_texture_format_BGRA8888 GL_EXT_texture_rg GL_EXT_texture_type_2_10_10_10_REV GL_OES_fbo_render_mipmap GL_OES_element_index_uint GL_EXT_shadow_samplers GL_OES_texture_compression_astc GL_KHR_texture_compression_astc_ldr GL_KHR_texture_compression_astc_hdr GL_KHR_texture_compression_astc_sliced_3d GL_KHR_debug GL_EXT_occlusion_query_boolean GL_EXT_disjoint_timer_query GL_EXT_blend_minmax GL_EXT_discard_framebuffer GL_OES_get_program_binary GL_OES_texture_3D GL_EXT_texture_storage GL_EXT_multisampled_render_to_texture GL_OES_surfaceless_context GL_OES_texture_stencil8 GL_EXT_shader_pixel_local_storage GL_ARM_shader_framebuffer_fetch GL_ARM_shader_framebuffer_fetch_depth_stencil GL_ARM_mali_program_binary GL_EXT_sRGB GL_EXT_sRGB_write_control GL_EXT_texture_sRGB_decode GL_KHR_blend_equation_advanced GL_KHR_blend_equation_advanced_coherent GL_OES_texture_storage_multisample_2d_array GL_OES_shader_image_atomic GL_EXT_robustness GL_EXT_draw_buffers_indexed GL_OES_draw_buffers_indexed GL_EXT_texture_border_clamp GL_OES_texture_border_clamp GL_EXT_texture_cube_map_array GL_OES_texture_cube_map_array GL_OES_sample_variables GL_OES_sample_shading GL_OES_shader_multisample_interpolation GL_EXT_shader_io_blocks GL_OES_shader_io_blocks GL_EXT_tessellation_shader GL_OES_tessellation_shader GL_EXT_primitive_bounding_box GL_OES_primitive_bounding_box GL_EXT_geometry_shader GL_OES_geometry_shader GL_ANDROID_extension_pack_es31a GL_EXT_gpu_shader5 GL_OES_gpu_shader5 GL_EXT_texture_buffer GL_OES_texture_buffer GL_EXT_copy_image GL_OES_copy_image GL_EXT_color_buffer_half_float GL_EXT_color_buffer_float GL_EXT_YUV_target GL_OVR_multiview GL_OVR_multiview2 GL_OVR_multiview_multisampled_render_to_texture GL_KHR_robustness GL_KHR_robust_buffer_access_behavior GL_EXT_draw_elements_base_vertex GL_OES_draw_elements_base_vertex "
		===================================


  If the installation is Okay, you will see a spinning 3D cube.


# Kodi Media Center on RK3399

  Kodi has support for GBM and can run on our NanoPi M4 setup. Kodi 18b5 runs fine for the most basic formats, let see if we can get 18rc3 soon.
  Building instructions are provided here: https://github.com/xbmc/xbmc/blob/master/docs/README.Linux.md#4-build-kodi

  Tips to build Kodi:

  * Install GBM and libdrm on top of fbdev (mali-gbm user space)
  * Install VPU support (rkmpp)
  * Install ffmpeg (rkmpp)
  * Build Kodi

  Knowning before hand Kodi 18b5 is expected to work on RK3399 (thanks to user fosf0r), i then pícked and tried this version.
  **Update**: 
  I tried **Kodi 18rc3** from github but **Kodi** breaks on **DRM**, so we should stick with 18b5 for now.
  I tried to figure out what could be wrong on the DRM part, but nothing i could find relevant.
  The headers have been updated to build Kodi 18rc3 , this new headers could be the culprit and i am not an experienced 
  Kodi builder or Kodi user anyway.


Splash screen
![Kodi 1](https://github.com/avafinger/nanopi-m4-ubuntu-base-minimal/raw/master/kodi/screenshot000.png)

ScreenShot 1
![Kodi 1](https://github.com/avafinger/nanopi-m4-ubuntu-base-minimal/raw/master/kodi/screenshot001.png)

ScreenShot 2
![Kodi 1](https://github.com/avafinger/nanopi-m4-ubuntu-base-minimal/raw/master/kodi/screenshot002.png)

ScreenShot 3
![Kodi 1](https://github.com/avafinger/nanopi-m4-ubuntu-base-minimal/raw/master/kodi/screenshot003.png)

ScreenShot 4
![Kodi 1](https://github.com/avafinger/nanopi-m4-ubuntu-base-minimal/raw/master/kodi/screenshot004.png)

ScreenShot 5
![Kodi 1](https://github.com/avafinger/nanopi-m4-ubuntu-base-minimal/raw/master/kodi/screenshot005.png)

ScreenShot 6
![Kodi 1](https://github.com/avafinger/nanopi-m4-ubuntu-base-minimal/raw/master/kodi/screenshot006.png)


# Release v1.3

  * Kernel 4.4.167
  * Faster boot times (~7 secs)
  * gcc (Ubuntu 8.2.0-1ubuntu2~18.04) 8.2.0
  * All dependecies updated and installed for Kodi Multimedia Theather (18rc3) - RK3399 
  * eDP LCD was removed from tree, for some reason EDP-1 is the main output and HDMI-1 is second monitor
    this can be troube if you want to install a Graphic Desktop, you get a dual header even if eDP LCD is not attached.
  * Rootfs ready to deploy Kodi Multimedia Center giving your NanoPi M4 / NEO4 a full blown Media Center (hopefully ;)


  **Instructions:**


    	* Download files:



		wget $(curl -s https://api.github.com/repos/avafinger/nanopi-m4-ubuntu-base-minimal/releases | grep -oP '"browser_download_url": "\K(.*)(?=")' | grep v1.3)




    	* Flash Image



    		sudo ./flash_kodi_sd.sh /dev/sdX (or /dev/mmcblkY) where X is a letter from b,c.. and Y is a number from 0,1..)


