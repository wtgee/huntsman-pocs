#==============================================================================
# Dockerfile for an ARM Huntsman device.
#==============================================================================

# This can be replaced with the official POCS docker image in future...
FROM raspbian/jessie:latest

# Environment variables
ENV PANUSER=huntsman
ENV PANDIR=/var/$PANUSER
ENV DEBIAN_FRONTEND=noninteractive
ENV CONDA_SCRIPT=Berryconda3-2.0.0-Linux-armv7l.sh
ENV CONDA_URL=https://github.com/jjhelmus/berryconda/releases/download/v2.0.0/$CONDA_SCRIPT
ENV CONDA_PATH=/home/$PANUSER/berryconda3
ENV POCS=${PANDIR}/POCS
ENV PAWS=${PANDIR}/PAWS
ENV HUNTSMAN_POCS=${PANDIR}/huntsman-pocs

# Add the huntsman user
RUN useradd -m -d /home/$PANUSER $PANUSER && \
    mkdir $PANDIR && \
    chown $PANUSER $PANDIR && \
    echo dialout plugdev netdev users input spi i2c gpio | xargs -n 1 groupadd -f && \
    usermod -a -G dialout,plugdev,netdev,users,input,spi,i2c,gpio,sudo huntsman && \
    # Install packages
    apt-get -y upgrade && \
    apt-get update && \
    apt-get -y dist-upgrade && \
    apt-get -y install bzip2 git gcc sshfs sudo && \
    # Allow sudo without password
    echo "$PANUSER ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers

# Install Berryconda as PANUSER
USER $PANUSER
WORKDIR $PANDIR
RUN wget $CONDA_URL && \
    chmod +x $CONDA_SCRIPT && \
    ./$CONDA_SCRIPT -b -p $CONDA_PATH

USER root
RUN ln -s $CONDA_PATH/bin/conda /usr/local/bin/ && \
    ln -s $CONDA_PATH/bin/python3.6 /usr/local/bin/python && \
    ln -s $CONDA_PATH/bin/pip /usr/local/bin/ && \
    # Install python deps
    pip install --upgrade pip && \
    pip install setuptools --upgrade && \
    conda install astropy matplotlib requests scipy && \
    pip install netifaces pyro4 pyyaml pyserial && \
    # Install huntsman-pocs
    mkdir -p $PANDIR/logs && \
    cd $PANDIR && \
    git clone https://github.com/panoptes/POCS.git && \
    git clone https://github.com/AstroHuntsman/huntsman-pocs.git && \
    cd $POCS && \
    python setup.py develop --no-deps && \
    cd $HUNTSMAN_POCS && \
    python setup.py develop && \
    # Setup permissions for PANUSER
    chmod 1777 /tmp && \
    chown -R $PANUSER $PANDIR && \
    chown -R $PANUSER /home/$PANUSER

#==============================================================================

ADD startup.sh .
CMD ["sh", "./startup.sh"]
