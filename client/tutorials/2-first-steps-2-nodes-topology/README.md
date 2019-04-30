 
# First steps
### How to perform simple TL operations and payments

<br/>
<br/>

All operations in GEO Network are performed through the trust lines. <br/>
In this tutorial we will run 2 nodes in a common network, connect them with a trust line, and perform several payment operations.

<br/>

## Node toplogy

The overall topology we are going to create looks like this:

<p align="center">
  <img src="https://github.com/GEO-Protocol/Documentation/blob/master/client/tutorials/2-first-steps-2-nodes-topology/resources/topology.png">
</p>

In this topology, node `A` trusts node `B` **$100**, so node `B` is able to pay to node `A` up to $100. <br/>
At the same time, node `A` is not able to pay to node `B` because node `B` does not trust node `A`.

<br/>

## Step 1: Topology creation

Now we are going to launch the nodes into the docker's internal network. <br/>
Please refer to the [first tutorial](https://github.com/GEO-Protocol/Documentation/tree/master/client/tutorials/1-docker-initialisation) for details on how to run the container with a node.

Please open 2 terminals and perform the following commands in each one:

```bash
> sudo docker run -i -t geoprotocol/network-client-beta 172.17.0.2 # Node A

> sudo docker run -i -t geoprotocol/network-client-beta 172.17.0.3 # Node B
```

<p align="center">
  <img src="https://github.com/GEO-Protocol/Documentation/blob/master/client/tutorials/2-first-steps-2-nodes-topology/resources/1.png">

  <img src="https://github.com/GEO-Protocol/Documentation/blob/master/client/tutorials/2-first-steps-2-nodes-topology/resources/2.png">
</p>


Now we should ensure that 2 containers are running in the internal network:

```bash
> sudo docker network inspect bridge
```

<p align="center">
  <img src="https://github.com/GEO-Protocol/Documentation/blob/master/client/tutorials/2-first-steps-2-nodes-topology/resources/3.png">
</p>

As we can see, our 2 containers are running well and are allocating 2 internal IPv4 addresses: `172.17.0.2` and `172.17.0.3`. Please pay attention to which container is allocating which address. Your containers' IDs are different from those in the tutorial, so be patient!

**Warning:**
Please ensure you are starting node `A` (`172.17.0.2`) first. 
Otherwise the docker will assign it a different address, in which case you should change the host parameter that is passed into the `run` command.

**Warning:**
In case your docker environment uses a different network subnet — ensure the correct host parameter in `run` command.

<br/>

# Step 2: Opening communication channels

Communication channels are used by nodes to perform secure packet exchange. All trust line operations require a settled communication channel.

### Opening CC on node A

```bash
> curl -X POST "http://172.17.0.2:3000/api/v1/node/contractors/init-channel/?contractor_address=12-172.17.0.3:2000"
```

[`Open Communication Channel API Specs`](https://github.com/GEO-Protocol/Documentation/tree/master/client/api-http#open-communication-channel)

<p align="center">
  <img src="https://github.com/GEO-Protocol/Documentation/blob/master/client/tutorials/2-first-steps-2-nodes-topology/resources/4.png">
</p>

Node reports sucess as well as some additional parameters that are used for channel setup on the counterpart node: `channel_id` and `crypto_key`.

* `channel_id` — ID of the channel that should be set up on node `B`.
* `crypto_key` — **SECRET** key that must be **securely sent** to node `B`. This key will not be sent by the GEO Node through GEO Network. Please, be patient! Compromising this key leads to traffic disclosure and the ability for a third party to change the operations flow!

</br>

### Opening CC on node B

```bash
> curl -X POST "http://172.17.0.3:3000/api/v1/node/contractors/init-channel/?contractor_address=12-172.17.0.2:2000&contractor_id=0&crypto_key=78b2e9736fbdd7b53931b998a42c1ae9f3c4caf3e1f864c93650d2167428a553" -i
```

[`Open Communication Channel API Specs`](https://github.com/GEO-Protocol/Documentation/tree/master/client/api-http#open-communication-channel)

<p align="center">
  <img src="https://github.com/GEO-Protocol/Documentation/blob/master/client/tutorials/2-first-steps-2-nodes-topology/resources/5.png">
</p>

As well as the previous node, node `B` reports that it's OK and shares its internal CC details.
At this stage, the nodes have established a secret channel for communication.

<br/>

# Step 3: Open Trust Line (A 100 -> B)

```bash
> curl -X POST "http://172.17.0.2:3000/api/v1/node/contractors/0/init-trust-line/1/" -i
```

[`Init Trust Line API Specs`](https://github.com/GEO-Protocol/Documentation/tree/master/client/api-http#open-trust-line-tl)

<p align="center">
  <img src="https://github.com/GEO-Protocol/Documentation/blob/master/client/tutorials/2-first-steps-2-nodes-topology/resources/6.png">
</p>

After this command, both nodes have an empty trust line with pre-shared public keys (1024, by default). 
Node `A` has sent its own public keys to node `B`, and node `B` has sent its public keys to node `A`. You can refer to corresponding logs of the nodes (`/node/client/operations.log`) for details about how the key exchange was done and how the keys were transferred.

At this stage, the trust line is set to (Outgoing Trust Amount = 0, Incoming Trust Amount = 0, Balance = 0) on both nodes in `equivalent = 1`. We assume the current `equivalent_id` of $ == 1. For more details, please refer to this video:

<p align="center">
  <a href="http://www.youtube.com/watch?feature=player_embedded&v=ieZKustA2Hk" target="_blank"><img src="http://img.youtube.com/vi/ieZKustA2Hk/0.jpg" 
  alt="GEO Protocol: Introduction to Trustlines by Dima Chizhevsky" width="720" height="480" border="10" /></a>
<p/>

Finally, we need to set it equal to $100. <br/>
To be able to send even cents, we now need to open a Trust Line for `$100 * 100 cents == 10000 cents`.

```bash
> curl -X PUT "http://172.17.0.2:3000/api/v1/node/contractors/0/trust-lines/1/?amount=10000" -i
```

[`Set Outgoing Trust Line API Specs`](https://github.com/GEO-Protocol/Documentation/tree/master/client/api-http#set-outgoing-trust-line)

<p align="center">
  <img src="https://github.com/GEO-Protocol/Documentation/blob/master/client/tutorials/2-first-steps-2-nodes-topology/resources/7.png">
</p>

Now we can check if the Trust Line has been set correctly. <br/>
We can do the next command on node `A`:

```bash
curl -X GET "http://172.17.0.2:3000/api/v1/node/contractors/trust-lines/0/10/1/"
```

[`List Trust Lines API Specs`](https://github.com/GEO-Protocol/Documentation/tree/master/client/api-http#list-trust-lines)

<p align="center">
  <img src="https://github.com/GEO-Protocol/Documentation/blob/master/client/tutorials/2-first-steps-2-nodes-topology/resources/8.png">
</p>

And on node `B`:

```bash
curl -X GET "http://172.17.0.3:3000/api/v1/node/contractors/trust-lines/0/10/1/"
```

[`List Trust Lines API Specs`](https://github.com/GEO-Protocol/Documentation/tree/master/client/api-http#list-trust-lines)

<p align="center">
  <img src="https://github.com/GEO-Protocol/Documentation/blob/master/client/tutorials/2-first-steps-2-nodes-topology/resources/9.png">
</p>

As we can see, both nodes are keeping a valid trust line state.

<br/>

# Step 4: Payments

Now, when the trust line from node `A` to node `B` is open and ready to perform operations, node `B` is able to send payments to node `A`. Let's send, for example, $50. 

First of all, let's be sure how many assets we can transfer:

```bash
> curl -X GET "http://172.17.0.3:3000/api/v1/node/contractors/transactions/max/1/?contractor_address=12-172.17.0.2:2000"
```

[`Max Flows API Specs`](https://github.com/GEO-Protocol/Documentation/tree/master/client/api-http#max-flow-predicition)

<p align="center">
  <img src="https://github.com/GEO-Protocol/Documentation/blob/master/client/tutorials/2-first-steps-2-nodes-topology/resources/10.png">
</p>

As expected, max flow from node `B` to node `A` equals `10000`. <br/>
Now, let's send $50.

```bash
> curl -X POST "http://172.17.0.3:3000/api/v1/node/contractors/transactions/1/?contractor_address=12-172.17.0.2:2000&amount=5000"
```

[`Payment API Specs`](https://github.com/GEO-Protocol/Documentation/tree/master/client/api-http#payment)

<p align="center">
  <img src="https://github.com/GEO-Protocol/Documentation/blob/master/client/tutorials/2-first-steps-2-nodes-topology/resources/11.png">
</p>

As we can see, node `B` reported Transaction UUID, so the operation was successful and the current topology is now the following:

<p align="center">
  <img src="https://github.com/GEO-Protocol/Documentation/blob/master/client/tutorials/2-first-steps-2-nodes-topology/resources/topology2.png">
</p>

Now, let's check max. flow from node `B` to node `A`:

```bash
> curl -X GET "http://172.17.0.3:3000/api/v1/node/contractors/transactions/max/1/?contractor_address=12-172.17.0.2:2000"
```

[`Max Flows API Specs`](https://github.com/GEO-Protocol/Documentation/tree/master/client/api-http#max-flow-predicition)

<p align="center">
  <img src="https://github.com/GEO-Protocol/Documentation/blob/master/client/tutorials/2-first-steps-2-nodes-topology/resources/12.png">
</p>

It seems to be correct. Node `B` has used 50% of its trust from node `A`. </br>
Now let's check the backward max. flow: from node `A` to node `B`:

```bash
> curl -X GET "http://172.17.0.2:3000/api/v1/node/contractors/transactions/max/1/?contractor_address=12-172.17.0.3:2000"
```

[`Max Flows API Specs`](https://github.com/GEO-Protocol/Documentation/tree/master/client/api-http#max-flow-predicition)

<p align="center">
  <img src="https://github.com/GEO-Protocol/Documentation/blob/master/client/tutorials/2-first-steps-2-nodes-topology/resources/13.png">
</p>

As we can see, now node `A` **is able to send payments to node** `B` and clear `B's` debt towards it. <br/> Let's try to pay $30 from node `A` to node `B`:

```bash
> curl -X POST "http://172.17.0.2:3000/api/v1/node/contractors/transactions/1/?contractor_address=12-172.17.0.3:2000&amount=3000"   
```

[`Payment API Specs`](https://github.com/GEO-Protocol/Documentation/tree/master/client/api-http#payment)

<p align="center">
  <img src="https://github.com/GEO-Protocol/Documentation/blob/master/client/tutorials/2-first-steps-2-nodes-topology/resources/14.png">
</p>

Node `A` reports success as well! Awesome! <br/>
As we can see, the basic mechanics work well and now it is time to go on to more complex topologies and cases.
