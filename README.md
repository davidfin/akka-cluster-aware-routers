# akka-cluster-aware-routers

This project was developed as part of the advanced akka training course by LightBend. 

## Two types of Cluster Aware Routers: 

1. Group: router sends messages to specified path using actor selection (Look up routees on member nodes).
2. Pool: router that creates routees as child actors and deploys them on member nodes. 

Cluster aware routers increase throughput and availability. 

The configuration of cluster aware group routers: 
``` config
actor {
    provider = akka.cluster.ClusterActorRefProvider
    deployment {
      /game-engine/scoresRouter {
        nr-of-instances = 10 // Attention: Total max number!
        routees.paths   = ["/user/scores-repository"]
        router          = random-group
        cluster {
          allow-local-routees          = off // Default is on
          enabled                      = on
          use-role                     = scores-repository
        }
      }
    }
}
```

## Exercise

In this exercise we split off the ScoresRepository as a separate instance and connect by way of a group 
router using Akka cluster aware routing.

1. Access the scoresRepository using FromConfig in GameEngine.scala
```scala
context.actorOf(FromConfig.props(Props[ScoresRepository]), "scoresRouter")
```
2. Since scoresRepository is remote, messages can get lost in Tournament.scala 
```scala
private def running(updateScoreRetryCount:Int) : Receive = {
    case Game.GameOver(gameScores) => scores ++= gameScores
    case Terminated(game)          => onGameTerminated(game)
    case ScoresUpdated =>
      log.info("scores update successful")
      context.stop(self)
    case Status.Failure(t: TimeoutException) =>
      if (updateScoreRetryCount > numScoreUpdateRetries) {
        log.info(s"scores update failed $numScoreUpdateRetries times, giving up.")
        context.stop(self)
      }
      else {
        log.info(s"scores update failed $updateScoreRetryCount times, trying again ...")
        scoresRepository ? ScoresRepository.UpdateScores(scores) pipeTo self
        context.become(running(updateScoreRetryCount + 1))
      }
  }
```

### command aliases (ge, pr, sr, ge2, pr2, sr2 and sj)
```scala
ge  // runs the game engine on port 2551
pr  // runs the player registry on port 2552
sr  // runs the scores repository on port 2553
ge2 // runs the game engine on port 2554
pr2 // runs the player registry on port 2555
sr2 // runs the scores repository on port 2556
sj  // runs the shared journal on port 2550
```
