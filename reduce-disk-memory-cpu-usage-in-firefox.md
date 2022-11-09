# Reduce disk, memory and CPU usage in Firefox
## Type about:config in your address bar
Set the following options:
````
browser.cache.disk.enable to false  // Disable disk cache.
                            // Media, like youtube videos, will not be downloaded directly on your hard disk
browser.sessionstore.interval to 1800000  // Store the session every 30 min, recovers it if browser crash happens
                            // Default is 15 sec, which uses I/O ALOT!
OR
browser.sessionstore.resume_from_crash to false  // If you want to disable this feature completely!

extensions.pocket.enabled to false  // If you don't use built-in Pocket service

browser.sessionhistory.max_entries to 5
browser.sessionhistory.max_total_viewers to 5  // Storages the list of sites that you have visited recently
                            // The maximum number of URLs you can traverse purely through the Back/Forward buttons.
````

## Type about:preferences in your address bar
- General tab: Uncheck "Check your spelling as you type"
- General tab: Uncheck "Recommend extensions as you browse"
- General tab: Uncheck "Recommend features as you browse"
- Home tab: Change Homepage and new windows to Blank Page
- Home tab: Change New tabs to Blank Page
- Home tab: Uncheck "Web Search"
- Home tab: Uncheck "Shortcuts"
- Search tab: Uncheck "Provide search suggestions"
- Privacy & Security tab: History to Never remember history. Use private browsing
- Privacy & Security tab: Disable all telemetry options
