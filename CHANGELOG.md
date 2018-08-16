Most of the pipelines with Jenkins today are built using Jenkins shared of libraries. These shared libraries provides huge amount of power
to the users of the libs as well as to the adminstrators of these libs.
Jenkinsfiles are best when they are used for configuration and defining the stages of a pipeline only, while the real core logic
of implementing those steps resides in a shared set of libraries.
If you are a pipeline admin in your team or company , you can define a lot of controls in your shared set of liberaries and provide a 
huge flexibility to the users at the same time.
One of the common problem which is prevalant in software development is commiting the bad/unwanted/malicious code in git repository
by supassing the approval process. Even while a git repository can be secured by setting up the right controls in the git settings,
a user with higher git privilages can always commit code intentionally or unintentionally to the release/master branch.

In such event, shared liberaries can be used to catch such commits, flag the build and take other actions for example notify the user and his superior.
So that the team can take appropriate decisions like reverting the commit or fixing the permissions on their git repositories.

Here is how you can do it:

1)- Mandate an overall closure in your Jenkinsfile. For example in this case `integration` : 

![pasted_image_8_16_18__4_05_pm](https://user-images.githubusercontent.com/11368123/44235324-b631ed80-a16e-11e8-9d99-d31722ba9d74.png)
