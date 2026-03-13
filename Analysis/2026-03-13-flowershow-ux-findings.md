
```
## Friction Log
- [ ] [path tested] — [what you tried] — [what happened / what was missing]
```
![[auth-error-same-email-git-google.png]]
If i already have a github account logged in -> then try login in with the same email, but now as gmail login, it gives "OAuthAccountNotLinked" quick video showing bellow: 
**Transcript here for AI context:**
"Okay, quick video here. When I try to log in with the same account that I have on my GitHub, it doesn't work. So my GitHub is attached to dev.teodoro right here. But if I try to log in with Google account as the same email, it doesn't work. So this is my GitHub account that already have logged in. And if I try to Google login, it doesn't work. It gives all accounts not linked. If I try with a new account that doesn't have a GitHub login already, right here, it does work normally."
![[login-issue-github-google.mp4]]

**First onboarding video here:**
**video transcript for ai here:** "Okay, so this is the hardest thing to test automatically. So I just logged in. I'm going to walk through every screen on the new onboarding flow without any prior knowledge. So I'm not going to skip any steps and not read ahead. This is a fresh start. And I will try to publish something using the drag and drop option with the dashboard. And yeah, drop a markdown file. And after that, I will... yeah basically log every movement I tried to do and yeah if I got confused or anything and I didn't know what to do next so I just logged in on a fresh account and I landed on the dashboard and it has create your first site so let's see right here test one create a site nice I can start with a template can import files let me try it out dragging dropping something right here if it works it doesn't just open the file okay uh okay now clicking it does work let me try something out let me try just drag dropping on the button also doesn't work okay uh okay so this should work now yes okay nice go back go to site settings okay this way we already know um Let's try syncing with GitHub. So this account is not synced with GitHub and I already have one that is synced. So let's try something that will probably get a bug out of it. Okay, so should I just close this? Did it work? Flusha was updated to a billion account. Okay, so I guess I can close this. Okay, I'm going to update the page. Didn't seem to link it. I'm going to try again. Okay, so... Yeah, let me try something else. If I already have my GitHub login, will it sync with Google then on the other way around? So I'm going to log in with my normal account. Yeah, this one. And sync with GitHub. What does it do? Now it does sync with GitHub repository. I don't want to have a specific repository, but it works as a publication, right? it's going to be a blog post. Yeah. So it does work if you're already logged in with GitHub. Okay. Let's try it out other ways. Sorry, if you log in with Google and you already had logged in with GitHub, it doesn't work. The button off sync with GitHub. Let's try with CLI. Okay. So yeah, my first publish is complete. Run the same command, but what command? Which one? Didn't give me any instructions on the CLI. Okay, so let's try deleting the site and fresh starting. So as a first publication. So if you already created a publication on drag and drop or other places and tried the CLI, it doesn't work. So, fast. And then the CLI. Okay, now it pops up. Okay, so let's do it. So it gives an error. Let me try. I already had an old version, so let me try uninstalling it. Okay, let's try installing it again. Then logging in doesn't work. Or publishing at least, no? Okay. So it might be an issue that already had the CLI installed, another version, another name, possibly because we changed the names. But yeah, if somebody already had the old name of it, I got to try it out later. I will document what I did if I managed to make it work. I think it's because my local machine, it already had the publish. So let's try Obsidian. Install Obsidian plugin. Yes, let me see if we have docs. Yes, nice. So it does open the Obsidian plugin. Let me also update it. Generate a personal access token. Go to our tokens page. Okay, create a token. Copy, done. Options. Paste this guy right here. Test. Okay, so I need to make the name test. So... Let me go to the settings, plugins, edit, test. But what if I try naming another thing? So yeah, since I created the site here named test, and if I don't put the name test, let's say that I already have a different name on this configuration. So it would be blank, right? If I didn't read the test, will it break or not? So yeah, let's try it out. So let's publish. Yeah, waiting for content upload. So yeah, it's waiting for a test name. Let's try something else. So if I would go into a new repo, install the plugin, and then put the name test, now it should automatically update that one. So if I hit publish right here, it should get, yeah, exactly. So it keeps waiting on, the same name that you put on the website. So if I view the website, it's going to be correctly. Yeah, Shipments, it's this one. Yeah, exactly. So if the user doesn't go into the settings and put the same name as site name on the Obsidian plugin, it will be infinitely loading on the Obsidian part. So yeah, let me see. But it didn't break the other one. So this one also works. Yes, exactly. Okay, this video is already long enough. I'll do some more testings. But this was my first initial onboarding part. The CLI error, I think it's my error part because I already had installed on a previous time. So since it updated, changed the name, I got to uninstall the other one and install it correctly. So, yeah."
![[first-onboarding.mp4]]


on installing CLI ( it can be a issue on my windows machine/my old installation residual files, or anything honestly....) but i got errors when installing the CLI publish, as shown in this video:![[fixing-cli-path-name.mp4]]
Claude code said :
"● Two problems, two fixes:

  ---
  Problem 1 — Node version is too old

  You're on v22.18.0 but the CLI needs >=22.22.0. Update Node first:

  # If you have nvm-windows:
  nvm install 22.22.0
  nvm use 22.22.0
  node --version  # confirm 22.22.0

  No nvm? Download the latest Node 22 LTS from https://nodejs.org and reinstall.

  ---
  Problem 2 — publish not found in PATH

  The install succeeded but PowerShell can't find the binary. Try this immediately without
  touching Node:

  npx @flowershow/publish auth login

  If that works, it's a PATH issue. Fix it permanently:

  # 1. Find where npm puts global binaries
  npm config get prefix

  # 2. Check if that path is in your PATH
  $env:PATH -split ';' | Where-Object { $_ -like '*npm*' }

  # 3. If it's missing, add it for this session:
  $env:PATH += ";$(npm config get prefix)"

  # 4. Then try again:
  publish auth login

  ---
  Order of operations:

  1. Run npx @flowershow/publish auth login right now — this bypasses both issues
  2. Update Node to 22.22.0+
  3. Reinstall: npm install -g @flowershow/publish@latest
  4. Confirm publish auth login works without npx"
Video fixing here
![[fixing-cli-path-name.mp4]]
onboarding "publish sync" got me confused, because sync would be after already creating website, but i understood afterwards
![[cli-test-1.png|370]]
e.g: i created a site called "test" and got this on the onboarding CLI: "publish sync ./my-content --name test" i tried changing the  "--name test" -> "--name ideas" and it gave error as img shows: ![[change-name-cli-sync-site.png|674]]

after site creation -> on site settings -> i want to delete the site and simply:
double click the name -> ctrl C -> paste in the copied name into delete input field, it has a space " " at the end of it, just small annoying thing when wanting to delete the site ![[copy-name-delete-site.png]]![[copy-name-delete-site.mp4]]