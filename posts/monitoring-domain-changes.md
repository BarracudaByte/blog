# Continious Domain Monitoring - Fighting Domain Squatting

![Programmer](https://raw.githubusercontent.com/BarracudaByte/blog/refs/heads/main/images/extracting-pe-info.webp?raw=true)

As cyber security analyst it is not uncommon to triage newly registered domains that potentially impersonate your company. There are multiple vendors out there, that are more than happy to notify you for a small amount of money, of a newly registered domain that is potentially domain squatting yours. 

If the domain name indeed sounds very similar but nothing is hosted on the website yet, there is not much that anyone can do. At least until it actually hosts something like a trademarked logo that you can action on.

The problem often though is, once the alert for a newly registered domain comes in, vendors often don't provide a way of keeping an eye on it, in case its content changes and only notify you once it is registered. Luckily for us, we can observe the domain ourselves, with just a few lines of Python.

## How it Works

The idea we are going to implement to check for any domain changes, is pretty straight-forward :
1. The analyst can add a domain to a watchlist
2. A job runs every so often to check all the domains in the watchlist, and gets their DOM
3. The automation compares the DOM to its previous version, and if it has changed it will notify the analyst to review it
4. After the review, the analyst can keep or remove the domain from the watchlist

# Implementation

## 