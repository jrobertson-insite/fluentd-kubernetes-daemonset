ARG SERVERCORE_VERSION=ltsc2019

FROM mcr.microsoft.com/windows/servercore:${SERVERCORE_VERSION}

SHELL ["powershell", "-NoLogo", "-Command", "$ErrorActionPreference = 'Stop'; $ProgressPreference = 'SilentlyContinue';"]

RUN Set-ExecutionPolicy Bypass -Scope Process -Force; \
    iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))

RUN choco install -y ruby --version 2.4.2.2 --params "'/InstallDir:C:\'"; \
    choco install -y msys2 --params "'/NoPath /NoUpdate /InstallDir:C:\ruby24\msys64'"

COPY Gemfile 'C:\fluentd\install\Gemfile'
COPY Gemfile.lock 'C:\fluentd\install\Gemfile.lock'

RUN refreshenv; \
    ridk install 2 3; \
    'gem: --no-document' | Out-File -Encoding UTF8 -NoNewline -Force -FilePath 'C:\ProgramData\gemrc'; \
    gem install bundler; \
    bundle config build.certstore_c --with-cflags="-Wno-attributes"; \
    bundle config build.yajl-ruby --with-cflags="-Wno-attributes"; \
    bundle config build.oj --with-cflags="-Wno-attributes"; \
    bundle install --gemfile='C:\fluentd\install\Gemfile'; \
    gem sources --clear-all; \
    Remove-Item -Force C:\ruby24\lib\ruby\gems\2.4.0\cache\*.gem; \
    Remove-Item -Recurse -Force C:\ProgramData\chocolatey

COPY upgrade-gems.ps1 '/fluentd/install/upgrade-gems.ps1'

COPY plugins/. /fluentd/plugins/
COPY conf/. /etc/fluent

CMD ["powershell", "-command", "fluentd -p /fluentd/plugins"]
