# #### GTB/GWB on ubuntu desktop
# K. Riitters October 2024
#  Built on kasmweb/desktop:1.14-rolling  https://hub.docker.com/r/kasmweb/desktop/tags
#  Uses example from Tyson Swetnam's CyVerse VICE kasm-ubuntu 22.04: https://github.com/cyverse-vice/kasm-ubuntu/tree/main/22.04
#      From there, downloaded supporting files kasmvnc_defaults.yaml, sudoers, vnc_startup.sh
# The local docker build is: docker build -t imagename .
# The local docker run is: docker run -it --rm -p 9876:6901  imagename
# Point local browser to 127.0.0.1:9876
#
# A docker image is available on Docker hub:  https://hub.docker.com/r/kriitters/kasm_gtbgwb
# ########
FROM kasmweb/desktop:1.14.0-rolling
USER root
# copy over Swetnam's configuration files. These are not specific to Swetnam's CyVerse platform.
COPY kasmvnc_defaults.yaml /usr/share/kasmvnc/kasmvnc_defaults.yaml
COPY vnc_startup.sh /dockerstartup/vnc_startup.sh
COPY sudoers /etc/sudoers
RUN chmod +x /dockerstartup/vnc_startup.sh
# end of basic image configuration
# Set up Google Earth repo entry for using GTB export to kml option
#    https://ubuntuhandbook.org/index.php/2024/05/install-google-earth-ubuntu/
RUN mkdir /etc/apt/keyrings/
#    add GE public key
RUN wget -qO - https://dl-ssl.google.com/linux/linux_signing_key.pub | gpg --dearmor |  tee /etc/apt/keyrings/google-earth.gpg >/dev/null
#    add GE repo 
RUN sh -c 'echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/google-earth.gpg] https://dl.google.com/linux/earth/deb/ stable main" > /etc/apt/sources.list.d/google-earth.list'
RUN chmod -R 777 /etc/apt/*
# Set up GTB/GWB install
WORKDIR /tmp
RUN wget https://ies-ows.jrc.ec.europa.eu/gtb/GTB/gtb_amd64.deb
RUN wget https://ies-ows.jrc.ec.europa.eu/gtb/GWB/gwb_amd64.deb
# Install things
RUN apt update \ 
	&& apt upgrade -y  \
    && apt install apt-utils dialog gedit nomacs gnumeric thunar-archive-plugin google-earth-pro-stable /tmp/gtb_amd64.deb /tmp/gwb_amd64.deb -y  \
	# remove GE 32-bit stuff
	&& apt remove libegl-mesa0:i386 libgbm1:i386 libgl1-mesa-dri:i386 libglapi-mesa:i386 libglx-mesa0:i386 -y \
	&& apt autoremove -y \
    && apt clean 
RUN rm -f gtb_amd64.deb gwb_amd64.deb
# avoid GE double entry for later update
RUN rm /etc/apt/sources.list.d/google-earth-*.list /etc/apt/trusted.gpg.d/google-earth-*.gpg
# For now keep the CyVerse trick, it also works on local image
RUN mkdir /home/.gwb
RUN chown kasm-user:kasm-user /home/.gwb
# Switch to normal user
WORKDIR /home/kasm-user
USER kasm-user
# fix up GWB things for kasm-user
RUN  ln -s /home/.gwb /home/kasm-user/.gwb
RUN  touch /home/kasm-user/.gwb/EULA.txt
# assign some preferred applications... 
RUN xdg-mime default nomacs.desktop "image/jpeg"
RUN xdg-mime default nomacs.desktop "image/png"
RUN xdg-mime default nomacs.desktop "image/tiff"
RUN xdg-mime default mupdf.desktop "application/pdf"
# avoid nomacs asking for language preference on first start
RUN mkdir -p /home/kasm-user/.config/nomacs
COPY ["Image Lounge.conf", "/home/kasm-user/.config/nomacs/"]
RUN sudo chown kasm-user:kasm-user /home/kasm-user/.config/nomacs/'Image Lounge.conf'
RUN chmod 644 /home/kasm-user/.config/nomacs/'Image Lounge.conf'
# End of GTB/GWB installation
# ######
# Local docker image run (the internal 6901 port is mapped to external 9876)
# docker run -it --rm -p 9876:6901 <imagename>
# Point browser to 127.0.0.1:9876
ENTRYPOINT ["/dockerstartup/vnc_startup.sh"]
# dockerfile end