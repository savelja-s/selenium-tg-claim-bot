FROM python:3.11-slim

RUN apt-get update && \
    apt-get install -y wget curl gnupg2 ca-certificates xdg-utils libasound2-dev git python3-pip python3-venv zbar-tools gdebi-core unzip --no-install-recommends && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

RUN python3 -m venv /venv && \
    /venv/bin/pip install --upgrade pip wheel && \
    /venv/bin/pip install selenium Pillow pyzbar qrcode-terminal python-telegram-bot requests mitmproxy

RUN curl -fsSL https://deb.nodesource.com/setup_current.x | bash - && \
    apt-get install -y nodejs && \
    npm install -g pm2@latest

COPY ./docker/install_chrome /usr/src/install_chrome.sh
RUN chmod +x /usr/src/install_chrome.sh && \
    /usr/src/install_chrome.sh

# Copy the project files into the container
RUN ln -sf /usr/share/zoneinfo/Europe/Kyiv /etc/localtime
RUN echo "Europe/Kyiv" > /etc/timezone

RUN echo "alias ll='ls -lah'" >> ~/.bashrc && \
    echo "alias gs='git status'" >> ~/.bashrc && \
    echo "alias gc='git clone'" >> ~/.bashrc


RUN echo "PM2 version: $(pm2 --version)" && \
    echo "Python version: $(python3 --version)" && \
    echo "Node.js version: $(node --version)" && \
    echo "npm version: $(npm --version)" && \
    echo "Chromium/Chrome version: $(chromium-browser --version || google-chrome --version)" && \
    echo "Chromedriver version: $(chromedriver --version)"


# Create app directory
WORKDIR /usr/src/app

# Copy the shell scripts and make them executable
COPY ./docker/*.sh /usr/src/app/

# Create and activate a virtual environment within the app directory, then install Python packages
RUN python3 -m venv /usr/src/app/venv && \
    /usr/src/app/venv/bin/pip install --upgrade pip && \
    /usr/src/app/venv/bin/pip install wheel selenium Pillow pyzbar qrcode-terminal mitmproxy python-telegram-bot requests

# Copy the launch script and make it executable
RUN find /usr/src/app -type f -name "*.sh" -exec chmod +x {} \;

# Ensure the virtual environment's Python interpreter is in the PATH
ENV PATH="/usr/src/app/venv/bin:$PATH"

# Copy the PM2 ecosystem configuration file
COPY ./docker/ecosystem.config.js /usr/src/app/

# Ensure PM2 resurrects saved process list on startup and then run enable-proxy
#CMD ["sh", "-c", "pm2 resurrect && pm2-runtime start ecosystem.config.js"]
CMD ["pm2-runtime", "start", "/usr/src/app/ecosystem.config.js"]
