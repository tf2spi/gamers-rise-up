# Puppy Games Discord Hack

Multiple gaming Discord servers have been hacked
by malware authors who presented their program
as a game.

## Findings

* Game has a string containing a URL to a discord
  cdn that contains the file "UnityGameManager.dll"

* The UnityGameManager.dll is itself an SFX that
  contains an Electron app that will probably extract itself.

* This electron app itself has a lot of obfuscated Javascript
  that probably does all the bad malware things.
  
