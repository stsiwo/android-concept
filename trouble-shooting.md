# Trouble Shooting 

If you encounter bugs/erros, check this note and if you find a solution, list your solution in chronical order on the topic.


## Android Studio

### Emulators crashe my computer when starting my project

change emulated performance value to Software (not hardware or automated). some of emulators don't allow to change the value (esp with Play Store) so choose the different one.

ref: https://stackoverflow.com/questions/39584765/ubuntu-16-04-1-lts-crashes-when-starting-android-emulator

ref: https://stackoverflow.com/questions/44328225/cant-change-emulated-performance-of-avd-in-android-studio

### Automatically Add Components to Manifest?

Yes. create an activity from Project in Android Studio and choose existing Activity such as Empty Activity. this automatically add the activity in Manifest.

### A file corrupted before Git commit. Is there any way to recover the file?

right click the file and select _local history_. you might be able to get local history. 

