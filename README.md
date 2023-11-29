# TORCS-parallel

This repo contains a custom version of the public Torcs driving simulator available at [SourceForge](https://sourceforge.net/projects/torcs/). It has been used to train a RL agent along different tracks available.

The modification made for this project are listed below:

- The starting countdown has been removed to fasten training.
- A patch has been aplied to reset Torcs via action ('Meta' = 1)

This project also contains an `INSTALLATION_GUIDE.md` file, which describes the steps to follow in order to reproduce the changes made. It also contains some other customizations such as changing the default car for the bots. Check it out for more information!