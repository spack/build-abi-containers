FROM gcc as base
ENV DEBIAN_FRONTEND=noninteractive
WORKDIR /code

RUN apt-get update && apt-get install -y git curl python3-botocore python3-boto3
RUN git clone --depth 1 https://github.com/spack/spack /opt/spack
ENV PATH=/opt/spack/bin:$PATH
RUN echo "packages:" > packages.yaml &&\
    echo "  all:" >> packages.yaml &&\
    echo "    target: [x86_64]" >> packages.yaml

COPY spack/ /test-packages
RUN spack repo add /test-packages && \
    spack config add -f "packages.yaml" && \
    {% for package in packages %}{% for version in package.versions %}spack install --no-checksum --source {% if cache_only %}--cache-only{% endif %} --deprecated {{ package.name }}@{{ version }} && \{% endfor %}{% endfor %}
    printf "Finished installation attempts\n"

FROM ghcr.io/buildsi/{{ tester.name }}:{{ tester.version }}
COPY --from=base /opt/spack /opt/spack

WORKDIR /build-si/
ENV PATH=/opt/spack/bin:$PATH
COPY {{ test.config_basename }} /build-si/tests.yaml
COPY {{ tester.runscript }} /build-si/{{ tester.runscript }}
{% for bin in bins %}COPY {{ bin }} /usr/local/bin/{{ bin }}
{% endfor %}
RUN apt-get install -y time python3-dev python3-pip && \
    pip3 install pytest && \
    mkdir -p /results && chmod +x /build-si/{{ tester.runscript }} {% if bins %}{% for bin in bins %} && chmod +x /usr/local/bin/{{ bin }}{% endfor %}{% endif %}
ENTRYPOINT ["{{ tester.entrypoint }}", {% for arg in tester.args %}"{{ arg }}", {% endfor %}"/build-si/{{ tester.runscript }}"]
