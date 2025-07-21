# Google SecOps SOAR

![Programmer and Google SecOps Logo](https://raw.githubusercontent.com/BarracudaByte/blog/refs/heads/main/images/google_secops_soar.webp?raw=true)


Google has been building up its range of security products in the recent years mostly by buying other companies or their products and start integrating it into their own. Siemplify (now Google SecOps SOAR) was one of these acquisitions and in this blog post I will show some of it strengths and weaknesses.

## Background

Siemplify was founded back in 2015 in Isreal with the goal to automated security operations (which is likely the goal of every single SOAR tool). In 2022 Siemplify was then acquired by Google for $500m as part of Googles plan to widen its Cyber Security tool set.

At the time Google had already build its own SIEM tool **Chronicle**, so after the acquisition Google renamed its new SOAR tool to ChronicleSOAR to align them. Shortly after they then merged the tools together, and Google SecOps was born.

![Siemplify and SecOps SOAR](https://raw.githubusercontent.com/BarracudaByte/blog/refs/heads/main/images/siemplify_secops.webp?raw=true)


## Functionality

Google SecOps has the following main areas:

- Cases & Environments
- Playbooks & Views
- Code (Integrations, Actions, Jobs, & Connectors)


### Environments & Cases

Environments in Google SecOps SOAR were originally likely designed for MSPs to manage different companies and their different access. Inside the same company they can still be used to create different case queues but don't have as much importance. Setting up the environments as `DEV`, `QA`, and `PROD` is not very advisable, as SecOps SOAR doesn't have an easy way to promote items from one environment to another, as it wasn't designed for this use case. 

Cases are interestingly structured in SecOps SOAR. They have two views, one general and one that can be influenced by the playbook that is running. Why exactly there are two views, is not clear to me (in my experience analysts don't like to jump between different views) and especially given that the first view can't be customised depending on the case it doesn't seem to be very useful to me. 

Another noteworthy item about cases in Google SecOps SOAR is that there is no dedicated area for analyst notes. Analyst can write comments that then get posted in a separate tab called the case wall, but all automation comments are also getting posted there, making it not the best spot for notes. Alternatively, a custom view can be added to the view of the playbook but, as there is no support for this, it takes a little bit of development work. 

Lastly, the case queue allows analysts to filter for specific cases and shows all open cases. Sadly, a feature to group alerts into, for example, by priority or SLA is not possible.

### Playbook & Views

I have already slightly touched on the point of views in Google SecOps SOAR in the previous section. Views are attached to playbooks, if multiple playbooks are running on a case, then only the view of the first playbook will be shown. The customisation options for views are pretty extensive, and predefined widgets allow to quickly put together interesting information. Sadly, predefined widgets currently can't be created and only exist for integrations from the marketplace, which limits there usefulness if you have a lot of custom integrations.

Playbooks work in a simple drag-and-drop manner, where new tasks can be added to predefined slots. Every playbook has a trigger that sets the condition if the playbook should be run on a specific case. When a new case comes in, SOAR loops through each of the enabled playbooks and checks if the tirgger matches, when it finds a match it stops checking further playbooks for a better match. This can sometimes cause very unwanted behaviour, if a trigger is to broad and suddenly runs a playbook on a case that was supposed to run something else instead.

![Programmer and Google SecOps Logo](https://raw.githubusercontent.com/BarracudaByte/blog/refs/heads/main/images/secops_playbook.webp?raw=true)

Due to the predefined slots, Google SecOps SOAR limits the structure of playbooks and doesn't allow for complex flows and makes it in addition quite difficult to restructure a playbook or putting some tasks aside for later. 

### Code

The code in Google SecOps SOAR is structured into different parts, `managers` contain the general library code which `actions` then can call. `Actions` are then the tasks that can be used in playbooks. Besides that there are `jobs` and `connectors`. `Jobs` can, same like `actions` call functions from the manager, however, only functions from the manager in the same integration. For example setting up a job that queries information from VirusTotal and URLScan.io is not possible, as the code for both is in different integrations. The only way around this limitation is to setup the job to simply create a new case, which then attaches a playbook to run the actual logic. A bit tedious in my opinion for a simple job. 

Besides that, Google SecOps SOAR allows to quickly add `pip` packages in the settings of the integration and allows to return simple values or complex JSON from an `action`.

# Pros & Cons

In comparison to other SOAR tools, like Palo Altos XSOAR, Google SecOps still has a lot of areas it can improve on and grow. 

## Version Control 

As indicated at the start, environments were not designed to be used as development or production instance and SecOps SOAR doesn't come with a built-in version control. An attempt was made by Google to remediate that with the [staging mode](https://cloud.google.com/chronicle/docs/soar/respond/ide/test-integrations-in-staging-mode), however, this only applies to integrations and not playbooks. The only alternative is to use the [GitSync integration](https://cloud.google.com/chronicle/docs/soar/marketplace/power-ups/gitsync). This integration works in general, but the files for playbooks and integrations were not designed with `git` in mind, even a small change, will highlight everything in Git and makes it nearly impossible in a Pull Request to see what the actual change was. 

## Playbooks

Building playbooks in SecOps SOAR takes some practice. Instead of being able to darg and drop tasks anywhere you want and connect them, SecOps SOAR automatically connects tasks for you. This has the obvious advantage of being quicker than having to conenct tasks yourselves, on the downside it removes a lot of felxibility of creating more complex paths. 

For example, tasks can be run in parallel, but only a maximum of five, and it doesn't work to have multiple tasks in order run simulatniously with one other task.

Conditionals are also restricted and only allow five branches. To handle additional branches, another conditional attached to the else of the first one has to be used.

## Views

Views over an extensive way of customising what analyst see during triage. However, the more you customise the more you run into issues that you can't easily communicate between the view and the playbook. For example, if you want to have a note input field and a button to save the notes and use them in the playbook, you can't directly. The API allows you to implement some funny workaround to make something like this work by for example, posting the notes as a comment to the case wall, and have a asnyc task in the playbook wait for the comment. Does it wokr? Yes! But should there be a better way? Definitely! 

# Conclusion

Google SecOps SOAR has some potential but some of its inital design choices may be hard to ever work around. Like the fact that playbooks have predefined slots and you can't create a custom flow is probably difficult to change in the backend, similar can be said for having native git support. 

For small SOCs its simple setup will have its benefit, as it is very easy to get into and understand. If you have on the other hand a dedicated engineer on the team to write automations, they would likely prefer using some other tool. 

If Google manages to iron out some of the current issues of its SOAR tool and potentially integrates the SIEM better, Google SecOps SOAR may become a good product but that depends all on how they implement those changes.

