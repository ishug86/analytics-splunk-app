## Topic: Separation of duties in the Code Commits with your pipeline

Most of the pipelines with Jenkins today are built using Jenkins shared of libraries. These shared libraries provide huge amount of power to the users of the libs as well as to the administrators of these libs.

Jenkinsfiles are best when they are used for configuration and defining the stages of a pipeline only, while the real core logic of implementing those steps resides in a shared set of libraries.

If you are a pipeline admin in your team or company, you can define a lot of controls in your shared set of libraries and provide a huge flexibility to the users at the same time.

One of the common problems which is prevalent in software development is committing the bad/unwanted/malicious code in git repository by surpassing the approval process. Even while a git repository can be secured by setting up the right controls in the git settings, a user with higher git privileges can always commit code intentionally or unintentionally to the release/master branch.

In such event, shared libraries can be used to catch such commits, flag the build and take other actions for example notify the user and his superior.
So that the team can take appropriate decisions like reverting the commit or fixing the permissions on their git repositories.

# Here is an example how you can do it:

1)- Wrap your Jenkinsfile in a parent closure. For example in this case `integration` is the parent closure: 


![pasted_image_8_16_18__4_05_pm](https://user-images.githubusercontent.com/11368123/44235324-b631ed80-a16e-11e8-9d99-d31722ba9d74.png)

2)- Since all the pipelines steps are under now under this closure, Before executing any pipeline steps, the code inside `integration` closure will be executed. With this power now you can apply all your admin rules under this closure.

3)- To implement commit validation functionality , In shared libs, in `integration` block you can write this example code:- 

```
stage('Validating the git commit'){
  def validateCommitWithPullRequest() {
      Map validationFlags = commitValidator(userIntiatedJob)
      boolean isValid = true
      // ignore user initiated job in case they need to rebuild due to error not related to their codebase
      if (validationFlags.pullViolation) {
          error "Validation failed: This change was not made through a pull request. This behavior is prohibited in Release branch."
      } else if (validationFlags.approverViolation) {
          error "Validation failed: The approver and requester of the pull request is same person. This behavior is prohibited in Release branch."
      } else if (validationFlags.noApproverViolation) {
          error "Validation failed: No approval was found for the pull request. This behavior is prohibited in Release branch."
      }
  }
  
  def commitValidator(boolean userIntiatedJob) {
      def hasResults = true
      int pageNum = 1
      def result = [pullViolation: true, approverViolation: false, noApproverViolation: false]
      while (hasResults) {
          String prListUrl = "https://<giturl>/api/v3/repos/<orgname>/<reponame>/pulls?state=closed&sort=updated&direction=desc&per_page=30&page=${pageNum}"
          def prResponse = <call git api>
          if (hasResults) {
              // Match the PR commit with the valid commit
              def results = prResponse.find { pr -> pr.merge_commit_sha == <git commit commit sha> }
              if (results != null) {
                  result.pullViolation = false
                  def prRequester = results.user.login
                  // github approver check
                  def prUrl = results.url as String
                  def approverResponse = <call git approvals>
                  String approverData = approverResponse.getContent()
                  // find approved review by person other than requester
                  validGithubApproval = approverData.any { pr -> pr.user.login != prRequester && pr.state == 'APPROVED' }
                  // find approved review by requester
                  invalidGithubApproval = approverData.any { pr -> pr.user.login == prRequester && pr.state == 'APPROVED' }
                  break
              }
          }
          pageNum++
      }
      return result
  }
}  
```

4)- Outcome , In case of a commit without approval: 


# View On the pipeline: 
![image](https://user-images.githubusercontent.com/11368123/44236639-50943000-a173-11e8-8d04-4ca83cb69275.png)


# In the job console log:
![image](https://user-images.githubusercontent.com/11368123/44236754-b2549a00-a173-11e8-9bb5-ff0a059d3546.png)



The best part about these controls is that such controls forces the converstations of doing the right thing. We have seen a lot of developers to rectify the git settings properly after seeing such errors.

This is a very basic illustration of how you can use shared liberaries to enforce controls. There are many more use cases that you can implement through using the shared liberaries to basically redefine the software development culture in your team.
Be sure to catch my talk on Reimagine your pipeline: Deliver software with confidence at Jenkins World 2018. 
