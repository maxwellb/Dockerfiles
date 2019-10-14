FROM debian

RUN apt-get update \
 && apt-get upgrade -y \
 && apt-get install -y debootstrap

ARG suite=stable
ARG mirror=http://deb.debian.org/debian
ARG variant=minbase
ARG components="main,contrib"
ARG base_include=" "
ARG base_exclude=" "
ARG localtime="Etc/UTC"

### Try up to three (3) times.
RUN debootstrap --include="${base_include}" --exclude="${base_exclude}" --components="${components}" --variant=${variant} ${suite} /mnt ${mirror} \
 || debootstrap --include="${base_include}" --exclude="${base_exclude}" --components="${components}" --variant=${variant} ${suite} /mnt ${mirror} \
 || debootstrap --include="${base_include}" --exclude="${base_exclude}" --components="${components}" --variant=${variant} ${suite} /mnt ${mirror}

### Debootstrap may leave artifacts in case of errors.
RUN rm -rf /mnt/debootstrap

### Dockerify
RUN rm -rf /mnt/usr/sbin/policy-rc.d \
 && (echo -e '#!/bin/sh\nexit 101\n' >> /mnt/usr/sbin/policy-rc.d) \
 && chmod 755 /mnt/usr/sbin/policy-rc.d \
 && ln -sf /bin/true /mnt/sbin/start-stop-daemon \
 && if [ -e /mnt/usr/share/zoneinfo/${localtime} ]; then ln -sf /usr/share/zoneinfo/${localtime} /mnt/etc/localtime; fi \
 && true

### Any customizations
RUN true

ARG additional_source_1=""
ARG additional_source_2=""
ARG additional_source_3=""
ARG additional_source_4=""
RUN if [ "x"!="x${additional_source_1}" ]; then \
    chroot /mnt bash -c 'echo '"'"${additional_source_1}"'"' >> /etc/apt/sources.list.d/distribution.list' \
    ; fi \
 && if [ "x"!="x${additional_source_2}" ]; then \
    chroot /mnt bash -c 'echo '"'"${additional_source_2}"'"' >> /etc/apt/sources.list.d/distribution.list' \
    ; fi \
 && if [ "x"!="x${additional_source_3}" ]; then \
    chroot /mnt bash -c 'echo '"'"${additional_source_3}"'"' >> /etc/apt/sources.list.d/distribution.list' \
    ; fi \
 && if [ "x"!="x${additional_source_4}" ]; then \
    chroot /mnt bash -c 'echo '"'"${additional_source_4}"'"' >> /etc/apt/sources.list.d/distribution.list' \
    ; fi \
 && chroot /mnt bash -c 'apt-get update && apt-get upgrade -y' \
 && echo The target size is $(du -sh /mnt). Removing some files... \
 && chroot /mnt bash -c 'apt-get clean' \
 && chroot /mnt bash -c 'rm -rf /var/lib/apt/lists/*' \
 && echo The target size is $(du -sh /mnt).


#### The final image
FROM scratch

COPY --from=0 /mnt /

RUN     mkdir -p /srv/workdir && chown -R nobody:nogroup /srv/workdir
USER    nobody:nogroup
WORKDIR /srv/workdir

CMD [ "bash", "--login" ]

