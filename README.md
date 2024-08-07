

About
This project builts a multi-branch pipeline for the 'cowsay' app. The 'cowsay' app simply looks like this:

<img width="357" alt="screen shot 2018-03-20 at 10 45 11 pm" src="https://user-images.githubusercontent.com/8520661/37696081-290403f0-2c91-11e8-9611-2ee8cbbfe877.png">

Pipelines Flow:
1. For branch 'master':
  Build
  Publish image
  Deploy (with port 80 exposed)
  E2E test

2. For branch 'staging':
  Build
  Publish image
  Deploy (with port 3000 exposed)
  E2E test

3. For branches feature/*:
  Build
  Publish image
  Test (locally)
  Cleanup (stop & remove container)

4. For branch release/*:
  Accept parameters 'MAJOR', 'MINOR'. (For manual creation)
  Verify branch `release/{MAJOR.MINOR}` exists. (Otherwise create - simulating the real process)
  Pull sources and checkout `release/{MAJOR.MINOR}`.
  Calculate and set a 3-number version (`MAJOR.MINOR.PATCH`) in `version.txt`.
  Build (along with the amended `version.txt` file).
  Test.
  Release.
  Clean Up: `git tag` with the 3-number version (`MAJOR.MINOR.PATCH`) and push the tag to the remote repository.
