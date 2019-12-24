FROM nvidia/opengl:1.0-glvnd-runtime-ubuntu16.04 as nvidia
FROM vladimirivan/ihmc-valkyrie-simulator-scs:stable

COPY --from=nvidia /usr/local /usr/local
COPY --from=nvidia /etc/ld.so.conf.d/glvnd.conf /etc/ld.so.conf.d/glvnd.conf

ENV NVIDIA_VISIBLE_DEVICES=all NVIDIA_DRIVER_CAPABILITIES=all