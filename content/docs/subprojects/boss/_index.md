---
title: Boss
---

Code: [https://github.com/DRuggeri/labinator_chef](https://github.com/DRuggeri/labinator_chef)

Boss is the "brains" of the lab's network operations and, like the heart of the lab itself, is managed with automated configuration management tooling. It is a [Chef Infra cookbook](https://www.chef.io/products/chef-infra), which is familiar from years of use. If the heart of this project were the building of Labinator with automation, I would have learned in much greater depth [Ansible](https://github.com/ansible/ansible) because it is a bit more appropriate for managing network gear (like OpenWRT on the firewall).

It's very difficult to distill down into a small summary the number of lessons learned in the creation of this cookbook. The [README](https://github.com/DRuggeri/labinator_chef/blob/main/README.md) of the project captures the overal list of daemons that had to come together in order to function. The commit history of each recipe should show a good enough series of trial/error/learning over the course of the project.

Top of mind things for me to expand on later:
* Cheap USB -> HDMI dongles are windows specific?!
* Keeping desktop very slim and looking decent
* Firefox and marionette mode