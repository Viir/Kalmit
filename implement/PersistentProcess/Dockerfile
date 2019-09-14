# Build dotnet build image
FROM microsoft/dotnet:2.2-sdk AS build-env
WORKDIR /app

# Copy everything and build
COPY . ./
WORKDIR /app/PersistentProcess.WebHost
RUN dotnet publish -c Debug -o out

# Build runtime image
FROM microsoft/dotnet:2.2-aspnetcore-runtime AS deploy-env

#####################
#PUPPETEER RECIPE
#####################
# As found at http://www.hardkoded.com/blog/puppeteer-sharp-docker

# Install latest chrome dev package and fonts to support major charsets (Chinese, Japanese, Arabic, Hebrew, Thai and a few others)
# Note: this installs the necessary libs to make the bundled version of Chromium that Puppeteer
# installs, work.
RUN apt-get update && apt-get -f install && apt-get -y install wget gnupg2 apt-utils
RUN wget -q -O - https://dl-ssl.google.com/linux/linux_signing_key.pub | apt-key add - \
    && sh -c 'echo "deb [arch=amd64] http://dl.google.com/linux/chrome/deb/ stable main" >> /etc/apt/sources.list.d/google.list' \
    && apt-get update \
    && apt-get install -y google-chrome-unstable fonts-ipafont-gothic fonts-wqy-zenhei fonts-thai-tlwg fonts-kacst ttf-freefont \
    --no-install-recommends \
    && rm -rf /var/lib/apt/lists/*

# Add user, so we don't need --no-sandbox.
# same layer as npm install to keep re-chowned files from using up several hundred MBs more space    
# RUN groupadd -r pptruser && useradd -r -g pptruser -G audio,video pptruser \
#     && mkdir -p /home/pptruser/Downloads \
#     && chown -R pptruser:pptruser /home/pptruser

# Run everything after as non-privileged user.
# USER pptruser
#####################
#END PUPPETEER RECIPE
#####################

ENV PUPPETEER_EXECUTABLE_PATH "/usr/bin/google-chrome-unstable"


WORKDIR /kalmit

ENV APPSETTING_webAppConfigurationFilePath="/kalmit/web-app-config.zip"
ENV APPSETTING_processStoreDirectoryPath="/kalmit/process-store"

COPY --from=build-env /app/PersistentProcess.WebHost/out /kalmit/dotnet/
ENTRYPOINT ["dotnet", "/kalmit/dotnet/PersistentProcess.WebHost.dll"]

# Configure a default app. Replace this file with a zip archive which contains your app.
COPY ./default-config/web-app/web-app-config.zip /kalmit

# ENV APPSETTING_adminRootPassword="password-for-kalmit-root"
