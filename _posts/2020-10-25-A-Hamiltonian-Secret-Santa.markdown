---
layout: post
title: "A Hamiltonian Secret Santa"
date: 2020-10-25 00:00:00
tags: programming
excerpt: "Here's to you Mr Hamilton"
---

# Introduction

2 years ago, when celebrating Christmas with 10 friends, we decided to give gift to each others and organize a Secret Santa. For those how do not know what Secret Santa is, the concept is quite simple: before Christmas, everybody is attributed someone else to whom they'll gift a present. On Christmas Day, you pick someone to start who gifts their present then the receiver gifts their present to the right person and so on.

So, we decided to organize a Secret Santa but at our Christmas party, something went wrong: their was an inner loop in the distribution! I started and gave my present to person A. Person A gave their present to person B and person B gave their present to me. That meant that we had to pick someone else to start again and finish the gifting for the remaining 7 people.

![Example of bad Secret Santa](/assets/posts/secretsanta/badGraph.jpg)

Last year, I decided that I would not allow this to happen again. Let's see how we can write an application that picks secret Santas with the help of our best friend: maths.

# What maths has to say

## Graph theory

A fascinating field of mathematics is graph theory. It lets you represent and solve problems using graphs.

A graph is a structure, directed or not, that consists of vertices (also called nodes) and edges. A typical example is to use a graph to represent possible trips between geographical locations. In that case, the vertices are cities and the edges represent whether it's possible to travel directly from city A to city B. This is illustrated in the picture below.

![Cities represented as a graph](/assets/posts/secretsanta/cityGraph.jpg)

In this example, it's possible to go directly from Paris to Madrid but not from Paris to Berlin. To do so, you first need to go through Brussels. It is also possible to give a weight to the edges. In the example, it could represent the distance between cities. With that, we could try to find the shortest path from one city to another.

As mentioned before, a graph can be directed or not. The cities example is non-directed: there is an edge between Paris and Brussels, that means that it's possible to go from Paris to Brussels but also to come back from Brussels to Paris. We could imagine making that example directed, with which we could represent one-way streets for instance.

Very good, now that we've discussed a bit of theory, let's try and apply it to our problem. For Secret Santa, the nodes are the participants and the edges represent the act of giving a gift to someone. This means we'll need a directed graph since we want to know who is giving the gift and who's receiving it. Finally, we won't give any weight to our edge as we will almost either give a gift or none; we never give half a gift.

## Hamilton to the rescue

Several mathematicians looked into graph theory and the one that will be helpful here is William Rowan Hamilton. Indeed, a graph is called Hamiltonian if it contains an Hamiltonian cycle. An Hamiltonian cycle consists of 2 parts:

- Cycle: A path is a cycle if the edge from which the path start is also the one at which it ends
- Hamiltonian: A path is Hamiltonian if it goes through each node once and only once

The good news is that it's exactly what we're looking for! We want our path to include all participants and they should only give (and receive) one present. The bad news is that there isn't an algorithm which guarantees finding an Hamiltonian in polynomial time. Fortunately, our graph will usually not have a lot of constraints so in practice, it will be possible to usually find a solution in linear time.

# Implementation

## Graph representation

Before implementing our algorithm, let's see how we are going to represent our data. A typical method to represent a directed graph is to use an 2-D array `tab` where the value `tab[x][y]` represents whether an edge exists from x to y. In our case, we'll use two values:

- `POSSIBLE` if an edge can be drawn from x to y
- `IMPOSSIBLE` if there's no edge from x to y

This is illustrated in the array below, where a white cell represent the `POSSIBLE` state and a red cell the `IMPOSSIBLE` state.

![Table representation of the graph](/assets/posts/secretsanta/secretSantaTable.jpg)

The diagonal only contains impossible cells since nobody can be their own secret Santas. I've also added a constraint that two people in a relationship cannot be each other's Secret Santas so that we have more constraints (and I found it more fun). We could easily imagine adding more constraints. For example, if Donald was Pluto's Secret Santa last year, we could forbid it to happen again this year.

## Finding an Hamiltonian cycle

Let's now dive into the algorithm itself. As mentioned before, it's impossible to have an algorithm that guarantees a polynomial execution time but since our graph doesn't have many constraints and as I don't have that many friends, an exhaustive approach should do the trick.

Here is our approach:

1. Pick a node a random. Add it to the cycle. It's our starting node
2. For that node, pick randomly another reachable node that has not been visited yet
3. Add that new node at the end of the cycle:
- If it's the last node we needed to visit, we are done
- Else, repeat step 2 using the new node

![Example of good Secret Santa](/assets/posts/secretsanta/goodGraph.jpg)


## Java implementation

Let's have a look at an implementation of that algorithm in Java

```java
public class Participant {

    private String name;
    private List<String> exclusionList;
//...
}
```

```java
public class SecretSantaService {

    public enum Status {
        Possible,
        Impossible,
        ;
    }

    public List<Participant> computeHamiltonCycle(List<Participant> participants) {
        Map<Participant, Map<Participant, Status>> giftsTable = buildGiftsTable(participants);
        ArrayList<Participant> hamiltonCycle = new ArrayList<>();
        hamiltonCycle.add(participants.get(0));
        if (!computeHamiltonCycle(giftsTable, hamiltonCycle, participants.get(0))) {
            throw new IllegalArgumentException("Could not compute a proper secret santa based on provided participants");
        }

        return hamiltonCycle;
    }

    private Map<Participant, Map<Participant, Status>> buildGiftsTable(List<Participant> participants) {
        Map<Participant, Map<Participant, Status>> giftsTable = new HashMap<>();
        for(Participant givingParticipant : participants) {
            Map<Participant, Status> statusMap = new HashMap<>();
            for(Participant receivingParticipant : participants) {
                if (givingParticipant.getExclusionList() != null && givingParticipant.getExclusionList().contains(receivingParticipant.getName())) {
                    statusMap.put(receivingParticipant, Status.Impossible);
                } else {
                    statusMap.put(receivingParticipant, Status.Possible);
                }
            }
            giftsTable.put(givingParticipant, statusMap);
        }
        return giftsTable;
    }

    private boolean computeHamiltonCycle(Map<Participant, Map<Participant, Status>> giftsTable, List<Participant> hamiltonCycle, Participant givingParticipant) {
        if(hamiltonCycle.size() == giftsTable.keySet().size() && giftsTable.get(givingParticipant).get(hamiltonCycle.get(0)) == Status.Possible) {
            //We are done
            return true;
        }

        List<Participant> possibleReceivingParticipants = getPossibleReceivingParticipants(giftsTable.get(givingParticipant), hamiltonCycle);
        while(possibleReceivingParticipants.size() > 0) {
            int receivingParticipantIdx = getRandomNumberInRange(0, possibleReceivingParticipants.size() - 1);
            Participant receivingParticipant = possibleReceivingParticipants.get(receivingParticipantIdx);
            hamiltonCycle.add(receivingParticipant);
            if (computeHamiltonCycle(giftsTable, hamiltonCycle, receivingParticipant)) {
                return true;
            }

            hamiltonCycle.remove(receivingParticipant);
            possibleReceivingParticipants.remove(receivingParticipantIdx);
        }

        return false;
    }

    private List<Participant> getPossibleReceivingParticipants(Map<Participant,SecretSantaService.Status> receivingParticipants, List<Participant> hamiltonCycle) {
        return receivingParticipants.entrySet()
                .stream()
                .filter(e -> e.getValue() !=SecretSantaService.Status.Impossible && !hamiltonCycle.contains(e.getKey()))
                .map(Map.Entry::getKey)
                .collect(Collectors.toList());
    }

    private int getRandomNumberInRange(int min, int max) {

        if(min == max) {
            return min;
        }

        if (min > max) {
            throw new IllegalArgumentException("max must be greater than min");
        }

        Random r = new Random();
        return r.nextInt((max - min) + 1) + min;
    }
}
```

It all starts with a call to the method `computeHamiltonCycle(List<Participant> participants)` to which we give a list of Participants. A Participant consists of a name and an exclusion list (to exclude couples for example).

The method `computeHamiltonCycle(List<Participant> participants)` will first build our graph (`giftsTable`) based on the provided Participants list and will then call the method `computeHamiltonCycle(Map<Participant, Map<Participant, Status>> giftsTable, List<Participant> hamiltonCycle, Participant givingParticipant)`, providing it with the freshly-created graph, the Hamiltonian cycle (containing initially only the starting node) as well as the first participant from which the computation will start.

In that method, we first check that we didn't reach the end of our calculation. Of course, for the first iteration, it's useless but we'll see that this method will be called recursively and this will serve as a stopping condition.

After, the method picks randomly a reachable node that has not been visited yet and adds it to the cycle. It then calls itself  using the new node. If that calls fails (the method returns `false`), we remove the node from the end of the cycle and try with another one.

Finally, once the cycle is of the same length as the number of participants, we verify that it's possible to "close" the cycle by verifying that the last node is allowed to give a gift to the starting one. If it's the case, we are done and we have found our Hamiltonian cycle.

## Create a webservice

To share this service with the world so that nobody has to deal with non-optimal Secret Santas ever again, we can easily use Spring Boot and create a Controller calling our service.

```java
@Controller
@RequestMapping("/")
public class SecretSantaController {

    private final SecretSantaService secretSantaService;

    public SecretSantaController(SecretSantaService secretSantaService) {
        this.secretSantaService = secretSantaService;
    }

    @RequestMapping(method= RequestMethod.GET)
    public SecretSantaResponse getSecretSanta(SecretSantaRequest request) {
        var hamiltonCycle = secretSantaService.computeHamiltonCycle(request.getParticipants());
        List<SecretSanta> secretSantas = buildSecretSantas(hamiltonCycle);
        return new SecretSantaResponse(secretSantas);
    }

    private List<SecretSanta> buildSecretSantas(List<Participant> hamiltonCycle) {
        List<SecretSanta> secretSantas = new ArrayList<>();
        Participant previousParticipant = hamiltonCycle.get(hamiltonCycle.size() - 1);
        for (Participant participant : hamiltonCycle) {
            secretSantas.add(new SecretSanta(previousParticipant, participant));
            previousParticipant = participant;
        }
        return secretSantas;
    }
}
```

An example is available at [https://secretsanta.migwel.dev/](https://secretsanta.migwel.dev/).

## Test

To verify that it all works properly, let's also write some tests using JUnit.

```java
    @Test
    void testGetValidSecretSanta() throws IOException {
        InputStream inStream = this.getClass().getClassLoader().getResourceAsStream("validsecretsantarequest.json");
        SecretSantaRequest request = new ObjectMapper().readValue(inStream, SecretSantaRequest.class);
        SecretSantaResponse response = controller.getSecretSanta(request);
        List<SecretSanta> secretSantas = response.getSecretSantas();
        assertEquals(request.getParticipants().size(), secretSantas.size());

        Set<Participant> givingParticipant = new HashSet<>();
        Set<Participant> receivingParticipant = new HashSet<>();
        Participant previousReceiver = secretSantas.get(secretSantas.size() - 1).getReceiver();
        for(SecretSanta secretSanta : secretSantas) {
            assertEquals(previousReceiver, secretSanta.getSecretSanta());
            assertFalse(givingParticipant.contains(secretSanta.getSecretSanta()));
            assertFalse(receivingParticipant.contains(secretSanta.getReceiver()));
            givingParticipant.add(secretSanta.getSecretSanta());
            receivingParticipant.add(secretSanta.getReceiver());
            previousReceiver = secretSanta.getReceiver();
        }
    }
```

It's also important to test that if a request is invalid (for example, one of the participants doesn't want to give a gift to anybody), the call should fail.

```java
    @Test
    void testGetInvalidSecretSanta() throws IOException {
        InputStream inStream = this.getClass().getClassLoader().getResourceAsStream("invalidsecretsantarequest.json");
        SecretSantaRequest request = new ObjectMapper().readValue(inStream, SecretSantaRequest.class);
        assertThrows(IllegalArgumentException.class, () -> controller.getSecretSanta(request));
    }
```

# Conclusion

We are now ready to get into the holidays season without fear of Christmas inner loops. If you want to run your own instance, the source are available on [Github](https://github.com/Migwel/SecretSanta).
