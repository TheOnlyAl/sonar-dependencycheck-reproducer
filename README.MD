# Sonar Dependencycheck Reproducer

This is small reproducer project for a problem with [SonarQube](https://www.sonarsource.com/products/sonarqube/), [OWASP Dependency-Check](https://owasp.org/www-project-dependency-check/) and the [dependency-check-sonar-plugin](https://github.com/dependency-check/dependency-check-sonar-plugin).

The problem can be seen when you try verify this project using using `mvn clean verify`.
You will get the following error on the background task in Sonar:

```
org.sonar.ce.task.projectanalysis.component.VisitException: Visit of Component {key=de.adteligence.sandbox:sonar-dependencycheck-reproducer,type=PROJECT} failed
    at org.sonar.ce.task.projectanalysis.component.VisitException.rethrowOrWrap(VisitException.java:44)
    at org.sonar.ce.task.projectanalysis.component.VisitorsCrawler.visit(VisitorsCrawler.java:71)
    at org.sonar.ce.task.projectanalysis.step.ExecuteVisitorsStep.execute(ExecuteVisitorsStep.java:51)
    at org.sonar.ce.task.step.ComputationStepExecutor.executeStep(ComputationStepExecutor.java:79)
    at org.sonar.ce.task.step.ComputationStepExecutor.executeSteps(ComputationStepExecutor.java:70)
    at org.sonar.ce.task.step.ComputationStepExecutor.execute(ComputationStepExecutor.java:57)
    at org.sonar.ce.task.projectanalysis.taskprocessor.ReportTaskProcessor.process(ReportTaskProcessor.java:75)
    at org.sonar.ce.taskprocessor.CeWorkerImpl$ExecuteTask.executeTask(CeWorkerImpl.java:212)
    at org.sonar.ce.taskprocessor.CeWorkerImpl$ExecuteTask.run(CeWorkerImpl.java:194)
    at org.sonar.ce.taskprocessor.CeWorkerImpl.findAndProcessTask(CeWorkerImpl.java:160)
    at org.sonar.ce.taskprocessor.CeWorkerImpl$TrackRunningState.get(CeWorkerImpl.java:135)
    at org.sonar.ce.taskprocessor.CeWorkerImpl.call(CeWorkerImpl.java:87)
    at org.sonar.ce.taskprocessor.CeWorkerImpl.call(CeWorkerImpl.java:53)
    at com.google.common.util.concurrent.TrustedListenableFutureTask$TrustedFutureInterruptibleTask.runInterruptibly(TrustedListenableFutureTask.java:131)
    at com.google.common.util.concurrent.InterruptibleTask.run(InterruptibleTask.java:76)
    at com.google.common.util.concurrent.TrustedListenableFutureTask.run(TrustedListenableFutureTask.java:82)
    at java.base/java.util.concurrent.Executors$RunnableAdapter.call(Unknown Source)
    at java.base/java.util.concurrent.FutureTask.run(Unknown Source)
    at java.base/java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.run(Unknown Source)
    at java.base/java.util.concurrent.ThreadPoolExecutor.runWorker(Unknown Source)
    at java.base/java.util.concurrent.ThreadPoolExecutor$Worker.run(Unknown Source)
    at java.base/java.lang.Thread.run(Unknown Source)
Caused by: java.lang.IllegalStateException: Fail to process issues of component 'de.adteligence.sandbox:sonar-dependencycheck-reproducer'
    at org.sonar.ce.task.projectanalysis.issue.IntegrateIssuesVisitor.visitAny(IntegrateIssuesVisitor.java:107)
    at org.sonar.ce.task.projectanalysis.component.TypeAwareVisitorWrapper.visitAny(TypeAwareVisitorWrapper.java:77)
    at org.sonar.ce.task.projectanalysis.component.VisitorsCrawler.visitNode(VisitorsCrawler.java:114)
    at org.sonar.ce.task.projectanalysis.component.VisitorsCrawler.visitImpl(VisitorsCrawler.java:97)
    at org.sonar.ce.task.projectanalysis.component.VisitorsCrawler.visit(VisitorsCrawler.java:69)
    ... 20 more
Caused by: java.lang.NullPointerException: Cannot invoke "java.util.Collection.stream()" because "this.bases" is null
    at org.sonar.core.issue.tracking.Tracking.getUnmatchedBases(Tracking.java:73)
    at org.sonar.core.issue.tracking.AbstractTracker.match(AbstractTracker.java:41)
    at org.sonar.core.issue.tracking.AnticipatedTransitionTracker.track(AnticipatedTransitionTracker.java:34)
    at org.sonar.ce.task.projectanalysis.issue.TransitionIssuesToAnticipatedStatesVisitor.onIssue(TransitionIssuesToAnticipatedStatesVisitor.java:72)
    at org.sonar.ce.task.projectanalysis.issue.IssueVisitors.onIssue(IssueVisitors.java:43)
    at org.sonar.ce.task.projectanalysis.issue.IntegrateIssuesVisitor.processIssue(IntegrateIssuesVisitor.java:211)
    at org.sonar.ce.task.projectanalysis.issue.IntegrateIssuesVisitor.lambda$processIssues$1(IntegrateIssuesVisitor.java:165)
    at java.base/java.lang.Iterable.forEach(Unknown Source)
    at org.sonar.ce.task.projectanalysis.issue.IntegrateIssuesVisitor.processIssues(IntegrateIssuesVisitor.java:165)
    at org.sonar.ce.task.projectanalysis.issue.IntegrateIssuesVisitor.visitAny(IntegrateIssuesVisitor.java:101)
    ... 24 more
```

# How to run this Project?

1. You need a running SonarQuebe Server with the dependency-check-sonar-plugin
2. Create a Token which can be used to do the analysis
3. Go into the pom.xml file and set/change the `sonar.host.url` and `sonar.token` value to your running server
4. Run `mvn clean verify` on the project root

# Tested with?

This project was tested with:

| Tool | Version |
| --- | --- |
| Maven | 3.9.6 |
| JDK | Eclipse Temurin 17.0.9 |
| SonarQube | Community Edition 10.4.1.88267 |
| dependency-check-sonar-plugin | 5.0.0 |

# Possible Workaround?

I did not find a suitable workaround but it seems that there is no problem when there are other issues on the code except dependency check issues. 

This, for example, can be checked by removing the `sonar.exclusions` from the pom.xml file.
Since the analysis of this will create an issue for `secrets:S6702`, and the background tasks runs without a problem.

# How to propably fix this?

I am not exactly sure what the main underlying problem is.
But it seems that the problem is mainly in the file: [TransitionIssuesToAnticipatedStatesVisitor.java](https://github.com/SonarSource/sonarqube/blob/10.4.1.88267/server/sonar-ce-task-projectanalysis/src/main/java/org/sonar/ce/task/projectanalysis/issue/TransitionIssuesToAnticipatedStatesVisitor.java).

The provided value of `anticipatedTransitions` in `public void onIssue(Component component, DefaultIssue issue)` seems to be `null` since it was not set in `public void beforeComponent(Component component)` because the component type is `PROJECT` and not `FILE`.

This could maybe be fixed by setting this value to an empty collection on init or when the parameter is set.