First run this to start the installer, I do it all headless because there is no X on my docker server:

    docker run -it \
        --device /dev/kvm \
        -p 50922:10022 \
        -p 5999:5999 \
        --name ventura \
        -e EXTRA="-display none -vnc 0.0.0.0:99,password=on" \
        -e GENERATE_UNIQUE=true \
        -e MASTER_PLIST_URL='https://raw.githubusercontent.com/sickcodes/osx-serial-generator/master/config-custom.plist' \
        sickcodes/docker-osx:ventura
    
Then you know when it is time to do something when things stop scrolling and you see  
audio: Failed to create voice `adc'  
hit enter and you should see the qemu prompt  
Now you have to type one of two things depending on your qemu version, either  
change vnc password  
or  
change vnc password user  

Then type in a password for vnc.  

Now you can connect to the installer, say your docker server is called dock, connect your vnc client to:  
vnc://dock:5999

First go in the disk manager and erase the big partition then exit disk manager when done.  
Then click reinstall mac os and go through whatever prompts to get it installed.  This takes a long time.  Make sure your docker.img is plenty big, this takes a lot of room.  If macos software update fails it means you need a bigger docker.img.  

Once all that is done you can enable screen sharing, this will let you vnc directly to the macos instead of the qemu vnc, which will give you copy/paste.   

Now shut down the VM and find the disk image that was created.  
find /var/lib/docker -size +10G | grep mac_hdd_ng.img  

Copy that somewhere and then go into the dir where you copied it.  This is where the macos disk will live.  

Then use the naked image to access it.  Here you can increase ram and cores.  Not sure if SMP is supposed to match CORES or what.  I give the container a name here with --name.  

    docker run -it \
        -e RAM=6 \
        -e SMP=2 \
        -e CORES=2 \
        --name ventura \
        --device /dev/kvm \
        -p 50823:10022 \
        -p 5998:5999 \
        -p 5990:5900 \
        -e EXTRA="-display none -vnc 0.0.0.0:99,password=on" \
        -v "${PWD}/mac_hdd_ng.img:/image" \
        sickcodes/docker-osx:naked
    
Make sure you removed the GENERATE_UNIQUE and MASTER_PLIST_URL options.  

Now connect to the macos vnc (if you enabled screen sharing) with your vnc client to  
vnc://dock:5990  
otherwise  
vnc://dock:5998  

Changes to the macos disk will be saved.  You can shut it down.  If you want to start it again find the docker id from docker ps or docker ps -a
and then connect to it like  
docker exec -it dockerIDhere bash  
once in the container shell type ./Launch-nopicker.sh  
to start qemu again and start the VM.  
