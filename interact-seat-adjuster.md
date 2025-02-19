# Interact with Seat Adjuster

We interact with the seat application through MQTT messages to emulate the behavior of an offboard application.
To see the responses from the seat adjuster app, subscribe to the MQTT topic `seatadjuster/#``.

```bash
mosquitto_sub -t 'seatadjuster/#' -v
```

To initiate the moving of the seat, publish an MQTT message for the seat adjuster application to set the position to `1000` (which is the equivalent of 100%):

```bash
mosquitto_pub -t seatadjuster/setPosition/request -m '{"position": 1000, "requestId": "12345"}'
```

You may need to open a second terminal session to send the message.
In the QEMU setup, you can do this through an SSH connection from the host machine by running `ssh -p 2222 root@localhost`.

After publishing the setPosition-message, the expected output for the `seatadjuster/#` subscription should look like this:

```bash
seatadjuster/setPosition/request {"position": 1000, "requestId": "12345"}
seatadjuster/setPosition/response {"requestId": "12345", "result": {"status": 0, "message": "Set Seat position to: 1000"}}
seatadjuster/currentPosition {"position": 00}
seatadjuster/currentPosition {"position": 10}
seatadjuster/currentPosition {"position": 20}
seatadjuster/currentPosition {"position": 30}
seatadjuster/currentPosition {"position": 40}
seatadjuster/currentPosition {"position": 50}
...
```

If you encounter an issue or want to look deeper into the state of the container, you can check the logs by executing:

```bash
kanto-cm --name <container-name> logs
```

## Run Kuksa Client

Besides trusting the log information and the MQTT messages that the seat position in the KUKSA Databroker has changed you can read the values
directly from the KUKSA Databroker with the KUKSA Client.
Through the `sdv-ctr-exec` it becomes possible to interact with the CLI of the Eclipse KUKSA Client:

```bash
kanto-cm create --i --t --network=host --name=kuksa-client ghcr.io/eclipse/kuksa.val/kuksa-client:master
kanto-cm start --name=kuksa-client
sdv-ctr-exec -n kuksa-client /kuksa-client/bin/kuksa-client --port 30555 --protocol grpc --insecure
```

Congratulations, you have reached the end of the seat adjuster guide. Based on the used code, you can now continue to realize other applications by using other signals.
You may need to adapt your `mock.py` accordingly.

## Deployment for Vehicle

To realize the described setup in a vehicle, you need to use a provider that is specific to the model and the covered VSS signals. As the picture below shows, this provider subscribes to changes in the relevant VSS signals in the Kuksa databroker. Upon changes, the provider interacts with the Seat ECU through the vehicle bus system, like a CAN bus. The ECU then controls the hardware, like the motor to adjust a seat.

![Leda Seat Adjuster Use Case](./img/seatadjuster-can.png)

A benefit of this setup is the split between vehicle-specific implementation and the general application.
Splitting these aspects makes it easier to port the application to other vehicles and save on duplicate realizations of the application logic.

At the same time, multiple applications interacting with the same VSS signal benefit from the re-use of the provider functionality instead of re-implementing the vehicle access again.

## References

- [https://github.com/COVESA/vehicle_signal_specification/blob/master/spec/Cabin/SingleSeat.vspec](https://github.com/COVESA/vehicle_signal_specification/blob/master/spec/Cabin/SingleSeat.vspec)
- [https://github.com/eclipse/kuksa.val.services/tree/main/seat_service](https://github.com/eclipse/kuksa.val.services/tree/main/seat_service)
- [https://eclipse.dev/velocitas/docs/about/use_cases/seat_adjuster/](https://eclipse.dev/velocitas/docs/about/use_cases/seat_adjuster/)
- [https://github.com/eclipse-velocitas/vehicle-app-python-sdk/tree/main/examples/seat-adjuster](https://github.com/eclipse-velocitas/vehicle-app-python-sdk/tree/main/examples/seat-adjuster)
- [https://github.com/eclipse-leda/meta-leda/blob/main/meta-leda-components/recipes-sdv/eclipse-leda/kanto-containers/example/seatadjuster-app.json.disabled](https://github.com/eclipse-leda/meta-leda/blob/main/meta-leda-components/recipes-sdv/eclipse-leda/kanto-containers/example/seatadjuster-app.json.disabled)
- [https://github.com/eclipse-leda/meta-leda/blob/main/meta-leda-components/recipes-sdv/eclipse-leda/kanto-containers/example/seatservice.json](https://github.com/eclipse-leda/meta-leda/blob/main/meta-leda-components/recipes-sdv/eclipse-leda/kanto-containers/example/seatservice.json)
