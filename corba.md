### CORBA (Common Object Request Broker Architecture) Overview

CORBA is a standard defined by the Object Management Group (OMG) to enable software components written in different languages and running on different platforms to work together. It is designed to facilitate the development of distributed, object-oriented systems.

### Key Components of CORBA

1. **Object Request Broker (ORB)**: The ORB is the middleware that establishes the client-server communication. It locates the object implementation to handle the client's request, regardless of the object's location or the languages used.

2. **Object Management Architecture (OMA)**: OMA is a framework for integrating software components. It leverages the ORB to execute distributed application tasks.

3. **Interface Definition Language (IDL)**: IDL is used to define the interfaces that objects present to the outside world. It is language-independent, allowing the definition of interfaces in a way that can be mapped to various programming languages.

### Example IDL

Here is an example of IDL defining interfaces for a hotel and guest room system:

```idl
interface GuestRoom; // forward declaration

interface Hotel {
    readonly attribute string name;
    readonly attribute short numberOfRooms;
    GuestRoom checkIn(in short numNights);
};

interface GuestRoom {
    readonly attribute short roomNumber;
    readonly attribute float rate;
    readonly attribute short numNights;
    void chargeMeal(in float amount);
    readonly attribute float balance;
    void checkOut();
};
```

### CORBA Client and Server Example

#### Client Implementation

The client locates and interacts with remote objects using the Naming Service. Here is a Java client example:

```java
import org.omg.CosNaming.*;
import Hotel.*;
import HotelHelper.*;
import GuestRoom.*;

public class HotelClient {
    public static void main(String[] args) {
        try {
            // Initialize the ORB
            org.omg.CORBA.ORB orb = org.omg.CORBA.ORB.init(args, null);

            // Get the Naming Service's root naming context
            org.omg.CORBA.Object obj = orb.resolve_initial_references("NameService");
            NamingContextExt rootContext = NamingContextExtHelper.narrow(obj);

            // Get the Hotel's object reference from the Naming Service
            obj = rootContext.resolve_str("Hotels/Hotel California");
            Hotel theHotel = HotelHelper.narrow(obj);

            if (theHotel == null) {
                System.err.println("ERROR: Object reference does not refer to a Hotel");
                System.exit(1);
            }

            // Invoke operations on the Hotel as if it were a local object
            String hotelName = theHotel.name();
            System.out.println("This is the Hotel " + hotelName);

            // Check into a room for 5 nights
            short numNights = 5;
            GuestRoom room = theHotel.checkIn(numNights);
            System.out.println("Balance at checkout for room " + room.roomNumber() + " will be: " + room.balance());
        } catch (Exception ex) {
            System.err.println("ERROR: " + ex);
            ex.printStackTrace(System.err);
        }
    }
}
```

#### Server Implementation

The server implements the IDL interfaces. Here is the implementation of the `Hotel` and `GuestRoom` interfaces:

```java
import Hotel.*;
import HotelHelper.*;
import HotelPOA.*;
import GuestRoom.*;
import GuestRoomHelper.*;
import GuestRoomImpl.*;
import org.omg.PortableServer.POA;
import org.omg.PortableServer.POAPackage.*;

public class HotelImpl extends HotelPOA {
    private static final int NUMBER_OF_ROOMS = 500;
    private GuestRoomImpl[] rooms = new GuestRoomImpl[NUMBER_OF_ROOMS];
    private boolean[] available = new boolean[NUMBER_OF_ROOMS];
    private POA guestRoomPOA;

    public HotelImpl(POA guestRoomPOA) {
        this.guestRoomPOA = guestRoomPOA;
        for (int i = 0; i < available.length; ++i) {
            available[i] = true;
        }
    }

    public String name() {
        return "Hotel California";
    }

    public short numberOfRooms() {
        return NUMBER_OF_ROOMS;
    }

    public GuestRoom checkIn(short numNights) {
        GuestRoom availableRoom = null;
        short nextRoom = 0;
        while (nextRoom < NUMBER_OF_ROOMS && !available[nextRoom]) {
            ++nextRoom;
        }

        if (nextRoom < NUMBER_OF_ROOMS) {
            available[nextRoom] = false;
            if (rooms[nextRoom] == null) {
                rooms[nextRoom] = new GuestRoomImpl(nextRoom);
            }
            rooms[nextRoom].checkIn(numNights);

            try {
                availableRoom = GuestRoomHelper.narrow(this.guestRoomPOA.servant_to_reference(rooms[nextRoom]));
            } catch (Exception ex) {
                System.err.println("ERROR: " + ex);
                ex.printStackTrace(System.err);
                return null;
            }
        }
        return availableRoom;
    }
}
```

#### GuestRoom Implementation

```java
import GuestRoom.*;
import GuestRoomHelper.*;
import GuestRoomImpl.*;
import org.omg.PortableServer.POA;

public class GuestRoomImpl extends GuestRoomPOA {
    private boolean checkedIn = false;
    private short roomNumber;
    private float rate = 100.0f;
    private short numNights;
    private float balance = 0.0f;

    GuestRoomImpl(short roomNumber) {
        this.roomNumber = roomNumber;
    }

    public void checkIn(short numNights) {
        this.checkedIn = true;
        this.numNights = numNights;
        this.balance = this.numNights * this.rate;
    }

    public short roomNumber() {
        return this.roomNumber;
    }

    public float rate() {
        return this.rate;
    }

    public short numNights() {
        return this.numNights;
    }

    public float balance() {
        return this.balance;
    }

    public void chargeMeal(float amount) {
        this.balance += amount;
    }

    public void checkOut() {
        this.checkedIn = false;
        this.numNights = 0;
        this.balance = 0.0f;
    }
}
```

#### Server Main Program

The main program initializes the ORB, creates a `Hotel` servant, registers it with the object adapter, and binds it into the Naming Service:

```java
import Hotel.*;
import HotelHelper.*;
import org.omg.CORBA.*;
import org.omg.PortableServer.*;
import org.omg.CosNaming.*;
import org.omg.CosNaming.NamingContextPackage.*;

public class HotelServer {
    public static void main(String args[]) {
        try {
            // create and initialize the ORB and POA
            ORB orb = ORB.init(args, null);
            POA poa = org.omg.PortableServer.POAHelper.narrow(orb.resolve_initial_references("RootPOA"));

            // create servant and register it with the POA
            HotelImpl servant = new HotelImpl(poa);
            byte[] hotelId = poa.activate_object(servant);

            // Get the Naming Service's root naming context
            org.omg.CORBA.Object obj = orb.resolve_initial_references("NameService");
            NamingContextExt rootContext = NamingContextExtHelper.narrow(obj);

            // Create a "Hotels" naming context
            try {
                rootContext.bind_new_context(rootContext.to_name("Hotels"));
            } catch (AlreadyBound e) {
                // consume exception
            }

            // Publish the Hotel's object reference to the Naming Service
            org.omg.CORBA.Object hotelObj = poa.id_to_reference(hotelId);
            rootContext.rebind(rootContext.to_name("Hotels/Hotel California"), hotelObj);

            // activate the RootPOA, and run
            poa.the_POAManager().activate();
            orb.run();
        } catch (Exception ex) {
            System.err.println("ERROR: " + ex);
            ex.printStackTrace(System.err);
        }
    }
}
```

### Conclusion

CORBA provides a robust framework for building distributed, object-oriented systems. By using IDL to define interfaces and ORB to handle communication, CORBA allows for seamless interaction between components written in different languages and running on different platforms. This tutorial covered the basics of CORBA, including IDL, client and server implementation, and the use of the Naming Service.
