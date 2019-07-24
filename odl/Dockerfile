FROM frinx/frinx_odl:4.2.1.frinx

ARG STORE_PASS
RUN echo ${STORE_PASS}
# copy updated distribution into container (if any available)
RUN mkdir /opt/distribution-frinx-override || true
COPY distribution/ /opt/distribution-frinx-override/
RUN if [ -f /opt/distribution-frinx-override/bin/karaf ]; then rm -R /opt/distribution-frinx && mv /opt/distribution-frinx-override /opt/distribution-frinx; fi

# copy licence to container image
COPY frinx.license.cfg /opt/distribution-frinx/etc

# add karaf features which to run at container startup
RUN ["sed", "-i", \
 "s/#odlFeaturesBoot.*/odlFeaturesBoot= \
  odl-restconf-all, \
  odl-netconf-topology, \
  frinx-cli-all-units, \
  frinx-unified-topology-all-units, \
  frinx-uniconfig-node-manager, \
  frinx-uniconfig-native, \
  frinx-lldp-topology /  \
", \
"/opt/distribution-frinx/etc/org.apache.karaf.features.cfg"]


# Configure
COPY odl-wrapper.sh /opt/distribution-frinx/bin/

# Configure TLS
RUN mkdir ./etc/store
WORKDIR /opt/distribution-frinx/etc/store

RUN keytool -keystore .keystore \
 -alias jetty \
 -genkey -keyalg RSA -storetype PKCS12 -validity 365 -keysize 2048 \
 -dname "O=Frinx, L=Bratislava, C=SK" \
 -storepass ${STORE_PASS}

WORKDIR /opt/distribution-frinx/etc
RUN sed -i 's/org\.osgi\.service\.http\.enabled=true/org\.osgi\.service\.http\.enabled=false/' "org.ops4j.pax.web.cfg"
RUN sed -i 's/org\.osgi\.service\.http\.enabled=true/org\.osgi\.service\.http\.enabled=false/' "org.ops4j.pax.web.cfg"

RUN sed -i 's/# org\.ops4j\.pax\.web\.ssl\.keystore=configuration\/ssl\/\.keystore/org\.ops4j\.pax\.web\.ssl\.keystore=etc\/store\/.keystore/' "org.ops4j.pax.web.cfg"

RUN sed -i 's/# org\.ops4j\.pax\.web\.ssl\.password=123456/'org\.ops4j\.pax\.web\.ssl\.password=${STORE_PASS}'/' "org.ops4j.pax.web.cfg"
RUN sed -i 's/# org\.ops4j\.pax\.web\.ssl\.keypassword=123456/'org\.ops4j\.pax\.web\.ssl\.keypassword=${STORE_PASS}/'' "org.ops4j.pax.web.cfg"
RUN cat org.ops4j.pax.web.cfg
WORKDIR /opt/distribution-frinx

ENTRYPOINT ["/bin/bash"]
CMD ["/opt/distribution-frinx/bin/odl-wrapper.sh"]
