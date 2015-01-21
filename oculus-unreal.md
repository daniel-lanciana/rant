## About

Recently participated in a 2-day hackathon where I worked solo and was lucky enough to get 3rd place!

> "Virtual Reality AWS stack viewer using Oculus Rift, Unreal Engine 4 and the AWS REST API."

## Thoughts

* Specs
 * Oculus Rift DK2
 * MacBook Pro (16Gb RAM)
* Oculus didn't work "out of the box"
 * Needed to rotate display 90 degrees
 * Recommended mirror mode, not extended display (resolution issues)
 * Recommended resolution is 1080p (1920x1080)
 * Some demos have a white 'bleeding' on the right side
* Setup
 * Unreal Engine ($20/mo subscription)
 * Oculus SDK (not required?)
 * Oculus runtime for OSX
 * XCode (3Gb)
* Issues
 * At one point Oculus would freeze laptop (spinning wheel of death) whenever plugged in and powered on. Resorted to safe mode to fix.
 * Unreal Engine crashes
 * VaRest plugin compiled for PC, had to find OSX somewhere. OSX version only compatible with 4.5.1 version of Unreal.
 * Locating the Unreal Engine directory (/Users/Shared/UnrealEngine/4.5) to manually install the VaRest plugin!
* Performance
 * Jarring motion
* Learning curve (navigation, placing objects, manipulating objects, perspective, blueprints) -- tutorial vids
* Deprecated UnrealScript support, only C++ in XCode?!
* Middleware EDDA for AWS JSON
* Could not use templates (e.g. Rift First Person) incompatable with older version of Engine (plugin). No backwards compatibility for maps.
* Importing models didn't apply nice textures (n00b)
* Failed attempt recompile plugin to 4.6
* Leveraged BestBuy project
* learnt to do TDD, presentation/end product first unless tech POC like me
* Rift not displaying correctly for my project?! Only when fullscreen...?
* Machine struggled a bit
* Simple tasks very long winded via Blueprint (e.g. REST call, random placement) -> these would get so huge! Reusable?
* Download demo, inputs not working -- hard to track down
* Reference viewer helpful
* SpawnVolume always run...hard to find
* ~stereo on
* Battling the editor: debugging, flow, 1 project open at a time, copying between projects, SpawnVolume.EventGraph hidden!
* Lack plugin support (VaRest only release candidate, poor docs)
* Blueprint sequence not working -- debug is bad, dummed down


https://github.com/alarial/ComputeMW2014-Discord/tree/master/Plugins/VaRest (project with OSX-compiled plugin)
https://github.com/Netflix/edda https://github.com/Netflix/edda/wiki/Configuration (EDDA)
https://forums.unrealengine.com/showthread.php?12874-VR-Game-Template (first person template)
http://api.remix.bestbuy.com/v1/categories(active=true)?format=json&apiKey=3x6v6vebcz7kdp5e7hdyg6kj
