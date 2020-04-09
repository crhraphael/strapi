#### 3. Install **Node.js** with **npm**:

Strapi currently supports `Node.js v12.x.x`. The following steps will install Node.js onto your EC2 server.

```bash
cd ~
curl -sL https://deb.nodesource.com/setup_10.x | sudo -E bash -
...
sudo apt-get install nodejs
...
node -v && npm -v
```

The last command `node -v && npm -v` should output two versions numbers, eg. `v12.x.x, 6.x.x`.

#### 4. Create and change npm's default directory.

The following steps are based on [how to resolve access permissions from npmjs.com](https://docs.npmjs.com/resolving-eacces-permissions-errors-when-installing-packages-globally):

- Create a `.npm-global` directory and set the path to this directory for `node_modules`

```bash
cd ~
mkdir ~/.npm-global
npm config set prefix '~/.npm-global'
```

- Create (or modify) a `~/.profile` file and add this line:

```bash
sudo nano ~/.profile
```

Add these lines at the bottom of the `~/.profile` file.

```ini
# set PATH so global node modules install without permission issues
export PATH=~/.npm-global/bin:$PATH
```

- Lastly, update your system variables:

```bash
source ~/.profile
```

You are now ready to continue to the next section.

#### 6. Install **PM2 Runtime**

Next, you need to install **PM2 Runtime** and configure the `ecosystem.config.js` file

[PM2 Runtime](https://pm2.io/doc/en/runtime/overview/?utm_source=pm2&utm_medium=website&utm_campaign=rebranding) allows you to keep your Strapi project alive and to reload it without downtime.

Ensure you are logged in as a **non-root** user. You will install **PM2** globally:

```bash
npm install pm2@latest -g
```

Now, you will need to configure an `ecosystem.config.js` file. This file will set `env` variables that connect Strapi to your database. It will also be used to restart your project whenever any changes are made to files within the Strapi file system itself (such as when an update arrived from Github). You can read more about this file [here](https://pm2.io/doc/en/runtime/guide/development-tools/).

- You will need to open your `nano` editor and then `copy/paste` the following:

```bash
cd ~
pm2 init
sudo nano ecosystem.config.js
```

- Next, replace the boilerplate content in the file, with the following:

```js
module.exports = {
  apps: [
    {
      name: 'your-app-name',
      cwd: '/home/ubuntu/my-strapi-project/my-project',
      script: 'npm',
      args: 'start',
      env: {
        NODE_ENV: 'production',
        DATABASE_HOST: 'your-unique-url.rds.amazonaws.com', // database Endpoint under 'Connectivity & Security' tab
        DATABASE_PORT: '5432',
        DATABASE_NAME: 'strapi', // DB name under 'Configuration' tab
        DATABASE_USERNAME: 'postgres', // default username
        DATABASE_PASSWORD: 'Password',
      },
    },
  ],
};
```

Use the following command to start `pm2`:

```bash
cd ~
pm2 start ecosystem.config.js
```

Your Strapi project should now be available on `http://your-ip-address:1337/`.

::: tip
Earlier, `Port 1337` was allowed access for **testing and setup** purposes. After setting up **NGINX**, the **Port 1337** needs to have access **denied**.
:::

#### 7. Configure **PM2 Runtime** to launch project on system startup.

Follow the steps below to have your app launch on system startup.

::: tip
These steps are based on the [PM2 Runtime Startup Hook Guide](https://pm2.io/doc/en/runtime/guide/startup-hook/).
:::

- Generate and configure a startup script to launch PM2, it will generate a Startup Script to copy/paste, do so:

```bash
$ cd ~
$ pm2 startup systemd

[PM2] Init System found: systemd
[PM2] To setup the Startup Script, copy/paste the following command:
sudo env PATH=$PATH:/usr/bin /usr/lib/node_modules/pm2/bin/pm2 startup systemd -u your-name --hp /home/your-name
```

- Copy/paste the generated command:

```bash
$ sudo env PATH=$PATH:/usr/bin /usr/lib/node_modules/pm2/bin/pm2 startup systemd -u your-name --hp /home/your-name

[PM2] Init System found: systemd
Platform systemd

. . .


[PM2] [v] Command successfully executed.
+---------------------------------------+
[PM2] Freeze a process list on reboot via:
   $ pm2 save

[PM2] Remove init script via:
   $ pm2 unstartup systemd
```

- Next, `Save` the new PM2 process list and environment.

```bash
pm2 save

[PM2] Saving current process list...
[PM2] Successfully saved in /home/your-name/.pm2/dump.pm2

```

- **OPTIONAL**: You can test to see if the script above works whenever your system reboots with the `sudo reboot` command. You will need to login again with your **non-root user** and then run `pm2 list` and `systemctl status pm2-ubuntu` to verify everything is working.
