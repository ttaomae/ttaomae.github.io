---
layout: post
title: Robot Testing
categories: software-engineering
tags: ics-691 java robocode testing
---
In a previous post I touched upon the idea of testing software, in the context of [FizzBuzz]({{ site.baseurl }}{% post_url 2013-01-16-fizzbuzz %}). Today, I will discuss testing once again. This time, instead of testing a simple program like FizzBuzz, I will be testing a more complex system. Specifically, I will be testing a Robocode robot.

## My Robot
Since I last discussed [Robocode]({{ site.baseurl }}{% post_url 2013-02-05-code-katas-robocode %}), I began working on a competitive robot. This time I've decided to use the `AdvancedRobot` class rather than the standard `Robot` class. Although I metioned in my last post that I would consider keeping track of all other robots, I have not yet implemented this. I decided to focus on a [1-vs-1](http://robowiki.net/wiki/One_on_One) robot rather than a [melee](http://robowiki.net/wiki/Melee) robot because it would be much simpler.

After a discussion on [composition over inheritence](http://en.wikipedia.org/wiki/Composition_over_inheritance) I reconsidered my previous approach to developing robots. For my code katas I decided to extend from a default robot which I implemented with additional functionality. However, now I've decided to use a utility class which would provide extra control over the robot. This would potentially allow me to have multiple utilities that could each provide different sets of functionality.

So far, my robot is still fairly simple. My basic strategy is to continuously circle the enemy. This constant movement will hopefully make it harder to aim at my robot. I will discuss my strategy in more detail when I describe how I tested my robot.

## Testing
Testing is a very important part of good software engineering. The most obvious advantage of testing is that it will help you verify that your system is operating correctly. For some systems, this can be a matter of life or death. This may sound like an exaggeration for software systems, however, if you consider that some medical equipment is controlled by software, it is not hard to imagine that poor testing can lead to disastrous results.

Good testing can also make it easier to develop larger and more complex systems. As you build larger and larger systems, it will become harder to pinpoint where a bug is occuring. Proper tests can help you detect when a component of your system breaks. The component that breaks will not always be the one that you are working on, which is why it can become very difficult to find bugs.

## Robot Testing
Robocode robots are a great way to show the value of testing. Sometimes a relatively small change to your robot can cause unexpected (and wrong) behavior. Testing will help you quickly detect and fix these mistakes.
To test my robot, I will be using three types of tests: acceptance tests, behavioral tests, and unit tests.

### Acceptance Tests
In general, acceptance tests are used to determine if a system meets its requirements. In this case, the requirements are that my robot can beat the following sample robots: Corners, Crazy, Fire, RamFire, SittingDuck, SpinBot, Tracker, Walls. For my tests, I assumed that "beating" a robot means that you score more points and that you are the last one to survive for at least 90 peceent of the battles. Writing acceptance tests will ensure that as I modify my robot it will still be able to beat each of these robots. This will help ensure that any changes to my strategy are not making my robot worse.

In order to write these acceptance tests, I first needed to download an additional file which would provide me with the `RobotTestBed` class. In order to use `RobotTestBed` you must extend it and specify which robots will be tested. It uses JUnit to run tests that you specify based on information that you can collect after each turn or after the battle ends.

To gain inspiration on how to use `RobotTestBed`, I looked at the code provided [here](https://github.com/philipmjohnson/robocode-pmj-dacruzer/tree/master/src/test/java/pmj).

Since I would need to provide several tests that would do the same thing except with different robots, I first created a base class which I would extend to specify which robots were battling.
Below I have provided the bulk of this base class.

```java
//package and imports omitted
/**
 * Tests that the test robot can beat the enemy robot at least 90% of the time.
 *
 * @author Todd Taomae
 */
public class TestBattleResults extends RobotTestBed {
  private String testRobot = "sample.SittingDuck";
  private String enemyRobot = "sample.SittingDuck";

  /**
   * Specifies the robots that are to be matched up in this test case.
   * @return The comma-delimited list of robots in this match.
   */
  @Override
  public String getRobotNames() {
    return this.testRobot + "," + this.enemyRobot;
  }

  /**
   * This test runs for 20 rounds.
   * @return The number of rounds.
   */
  @Override
  public int getNumRounds() {
    return 20;
  }

  /**
   * Returns true if the battle should be deterministic and thus robots will always start
   * in the same position each time.
   *
   * Override to return false to support random initialization.
   * @return True if the battle will be deterministic.
   */
  @Override
  public boolean isDeterministic() {
    return false;
  }

  /**
   * The actual test, which asserts that the test robot won at least 90% of the rounds
   * against the enemy robot.
   * @param event Details about the completed battle.
   */
  @Override
  public void onBattleCompleted(BattleCompletedEvent event) {
    // Return the results in order of getRobotNames.
    BattleResults[] battleResults = event.getSortedResults();

    BattleResults testRobotResults = battleResults[0];
    String testRobotName = testRobotResults.getTeamLeaderName();

    // check that the winner (first robot in sorted results) is the test robot
    assertTrue("Check " + this.testRobot + " winner", testRobotName.startsWith(this.testRobot));

    // check that the test robot won at least 90% of the battles
    assertTrue(testRobotName + " won " + testRobotResults.getFirsts() + " rounds",
        testRobotResults.getFirsts() > (int)(getNumRounds() * 0.9));
  }

  //getters and setters omitted
}
```

From here, it is simple to test any two robots to see if one can reliably beat the other. Below I have provided an example of how to do this.

```java
// package and imports omitted
/**
 * Tests that TestBot can beat EnemyBot 90% of the time.
 * @author Todd Taomae
 */
public class TestBattleResultsVersusCorners extends TestBattleResults {
  /**
   * Sets the names of the robots.
   */
  @Before
  public void setNames() {
    super.setTestRobot("example.TestBot");
    super.setEnemyRobot("example.EnemyBot");
  }
}
```

In the future, I may modify the base class so that the win percentage can be specified.

### Behavioral Tests
The next set of tests that I wrote were behavioral tests, which verify that my robot is behaving the way I intended. So far, I have written three tests. One for movement, one for aiming, and one for firing.

Besides simply circling the enemy, my robot also tries to stay at a certain distance. The idea is that you are more likely to hit a target that is closer. However, you do not want to go too close since it will also be easier for your opponent to hit you. My movement test will verify that my robot will stay at its preferred distance (+/- some buffer distance) for most of the battle.

In robocode, there are two basic parts to firing strategy. First there is targeting or aiming, and second is choosing your firing power. I've decided to test these two parts separately.

First, to test aiming, I selected the sample robot that does not move or shoot (SittingDuck) and made sure that nearly all of my bullets hit the enemy. The reason I do not test that all my bullets hit is because there will occasionally be a bullet that is fired before the enemy dies, but another bullet hits and kills the enemy before the last one can hit it. Although this does not prove the effectiveness of my aiming, it ensures that I do not have any bugs that might cause it to behave unexpectedly.

My firing strategy has two parts. First, I do not fire if I am farther than a specified distance. Second, I use less power if I am farther away. Both of these strategies help conserve energy. The second strategy also helps to improve accuracy. Bullets that use less power, travel faster so they will reach the target faster. To test the first strategy, I made sure that no bullets were fired if the two robots were farther than the maximum distance. To test the second strategy, I simply made sure that I fired bullets within three different ranges of power.

Keeping track of bullets proved a little more difficult than I expected because of the limited information that the `RobotTestBed` provides. The best you can do to track bullets is get a snapshot of all bullets that exist after each turn. Since bullets exist for more than one turn, you cannot simply go through all bullets after each turn because you will look at the same bullet multiple times. To solve this problem, I used a set to keep track of which bullets I already looked at. Below I have provided a snippet of code to show how I did this.

```java
/**
 * Finds when a bullet was first fired and does something with the bullet information.
 *
 * @author Todd Taomae
 */
public class TestBullets extends RobotTestBed {
  /** Set containing the ID of all bullets that have been fired. */
  private Set<Integer> bullets = new HashSet<Integer>();

  @Override
  public void onTurnEnded(TurnEndedEvent event) {
    IBulletSnapshot[] currentBullets = event.getTurnSnapshot().getBullets();

    for (IBulletSnapshot bullet : currentBullets) {
      // if the bullet was not previously in the set, this is the first turn that it exists
      if (!this.bullets.contains(bullet.getBulletId())) {
        // add it to the set so we know we looked at it
        this.bullets.add(bullet.getBulletId());

        // do something with the bullet information
      }
    }
  }
```

### Unit Tests
The last type of test that I did was a unit test. Unit tests, as the name implies, tests a single unit or component of a system. In this case, I tested my `MathUtility` class, which I use to calculate certain values commonly used in Robocode, such as the distance between two points. Since I know comparing floating point numbers for equality is not as simple as it seems, I did some research and came across [this StackOverflow post](http://stackoverflow.com/questions/5923682/comparing-floating-point-numbers-in-java/5923720#5923720) which describes a common method, using JUnit, to test floating point numbers.
Below I have provided a small section of my code which uses the described method.

```java

  /**
   * Tests the getDistance method using Pythagorean triples.
   */
  public void testGetDistancePythagoreanTriples() {
    double actual;
    double expected;

    actual = MathUtility.getDistance(0.0, 0.0, 3.0, 4.0);
    expected = 5.0;
    assertEquals(expected, actual, 5 * Math.ulp(actual));

    actual = MathUtility.getDistance(0.0, 0.0, 5.0, 12.0);
    expected = 13.0;
    assertEquals(expected, actual, 5 * Math.ulp(actual));

    actual = MathUtility.getDistance(0.0, 0.0, 7.0, 24.0);
    expected = 25.0;
    assertEquals(expected, actual, 5 * Math.ulp(actual));
  }
```

## Reflection
Writing these tests have proved a valuable experience for me. Although it was not always easy to write the tests, this exercise demonstrated the importance and usefulness of testing. On more than one occasion, while modifying my robot, I found that it was no longer passing some of the acceptance and behavioral tests. Having tests, quickly alerted me to these errors and because I tested my `MathUtility` class, I knew that this was not the source of errors, so I could ignore it while trying to find the bug. Further tests will no doubt be invaluable while I improve my robot and make it more complex.

