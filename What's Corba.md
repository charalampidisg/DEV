### Common Object Request Broker Architecture (CORBA)

Common Object Request Broker Architecture designs and implements distributed, object-oriented systems (CORBA). 800 global organisations joined the Object Management Group to create CORBA (OMG). OMG was founded in 1989 to promote scalable, reusable, and cheaper computer systems using open standards. Anyone may view the OMG’s specs.

#### Object Request Broker

Object Request Broker handles client-object communication (ORB). A client’s object implementation may be on the same or a separate host. The ORB finds a ready object implementation process to satisfy requests. Client and object implementations may employ separate hardware, OS, and languages. OMG defines standard COBOL language bindings.

#### Object Management Architecture

CORBA defines OMA as a software component integration architecture (OMA). Because of its ORB base, the OMA can execute dispersed application tasks.

Banking, healthcare, and telecommunications have specified services and facilities. OMG and its members define CORBAservices and CORBAfacilities, and vendors implement them. Multiple vendors’ Services and Facilities may communicate. Few CORBA applications leverage many of its services and features.

#### Interface Definition Language

IDL describes the client-server contract (IDL). IDL specifies the interface and operation parameters. IDL is independent of client/server languages. The syntax of IDL is similar to the syntax of Java or C++. IDL contains no implementation details, however, as IDL interfaces are implemented in conventional programming languages.

The IDL interfaces for our example are as follows:

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

This example has two interfaces: Hotel and GuestRoom.

Checking into the hotel gets a guest a GuestRoom. Once checked in, a guest may charge meals, check the room’s balance, and leave. An IDL Compiler creates client stub code and server skeleton code in the target language from the IDL code.

The stub code conceals the client from distant operation specifics. The skeleton code conceals the server from the low-level complexities of making a client-side implementation object accessible.

Our IDL compiler generates Java. If the client and server are written in separate languages, we’d compile the IDL code for each language.

Most IDL compilers let the programmer store produced Java code in a Java package. Each IDL compiler does things differently. Open-source JacORB is an example. “-p” specifies a package for JacORB’s IDL compiler. All created Java code goes into the `com.ociweb` package.

```sh
java org.jacorb.idl.parser -p com.ociweb Hotel.idl
```

### Clients And Servers

We’ll create a client and server using the Hotel and GuestRoom APIs.

Client gets Hotel’s object reference first. A client uses an object reference to trigger server object actions. Client and server broadcast and get object references using CORBA Naming Service.

The Naming Service links human-readable names to object references. Some of a server’s objects are bound to the Naming Service. Our server registers its Hotel but not its GuestRoom in Naming Service.

The `checkIn()` function returns GuestRoom.

#### The Client

The client must locate and invoke actions on distant objects. The customer utilises the Naming Service to locate a Hotel and a GuestRoom. It uses Hotel and GuestRoom as local Java references.

```java
import org.omg.CosNaming.*;
import Hotel.*;
import HotelHelper.*;
import GuestRoom.*;

public class HotelClient {
    public static void main(String[] args) {
        try {
            // Initialize the ORB.
            org.omg.CORBA.ORB orb = org.omg.CORBA.ORB.init(args, null);

            // Get the Naming Service’s root naming context
            org.omg.CORBA.Object obj = orb.resolve_initial_references("NameService");
            NamingContextExt rootContext = NamingContextExtHelper.narrow(obj);

            // Get the Hotel’s object reference from the Naming Service
            obj = rootContext.resolve_str("Hotels/Hotel California");
            Hotel theHotel = HotelHelper.narrow(obj);

            if (theHotel == null) {
                System.err.println("ERROR: Object reference does not refer to a Hotel");
                System.exit(1);
            }

            // Invoke operations on the Hotel as if it were a local object.
            // Note that “name” attribute is accessed via “name()” method
            String hotelName = theHotel.name();
            System.out.println("This is the Hotel " + hotelName);

            // Check into a room for 5 nights.
            short numNights = 5;
            GuestRoom room = theHotel.checkIn(numNights);
            System.out.println("Balance at checkout for room " + room.roomNumber() + " will be: " + room.balance());
        } catch (org.omg.CORBA.SystemException ex) { // Catch exceptions
            System.err.println("ERROR: " + ex);
            ex.printStackTrace(System.err);
        } catch (org.omg.CORBA.UserException ex) {
            System.err.println("ERROR: " + ex);
            ex.printStackTrace(System.err);
        }
    }
}
```

Java the complete Java developer course will enhance your knowledge and skills.

#### The Server

Server implements client IDL interfaces.

Hotel and GuestRoom are server interfaces.

We write Hotel’s implementation class. The servant class extends the IDL compiler’s HotelPOA class and implements its attributes, operations and other information.

IDL’s skeleton class is HotelPOA.

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
        // Determine if a room is available.
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
            } catch (ServantNotActive ex) {
                // Will not be thrown in this application
                System.err.println("ERROR: " + ex);
                ex.printStackTrace(System.err);
                return null;
            } catch (WrongPolicy ex) {
                // Will not be thrown in this application
                System.err.println("ERROR: " + ex);
                ex.printStackTrace(System.err);
                return null;
            }
        }
        return availableRoom;
    }
}
```

And here’s a servant class for GuestRoom:

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

Then, our server needs a main program.

The main program initialises the ORB, creates a Hotel servant, registers the Hotel servant with the object adapter, and binds the Hotel into the Naming Service:

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

            // Get the Naming Service’s root naming context
            org.omg.CORBA.Object obj = orb.resolve_initial_references("NameService");
            NamingContextExt rootContext = NamingContextExtHelper.narrow(obj);

            // Create a “Hotels” naming context
            try {
                rootContext.bind_new_context(rootContext.to_name("Hotels"));
            } catch (AlreadyBound e) {
                // consume exception
            }

            // Publish the Hotel’s object reference to the Naming Service
            org.omg.CORBA.Object hotelObj = poa.id_to_reference(hotelId);
            rootContext.rebind(rootContext.to_name("Hotels/Hotel California"), hotelObj);

            // activate the RootPOA, and run
            poa.the_POAManager().activate();
            orb.run();
        } catch (org.omg.CORBA.SystemException ex) {
            System.err.println("ERROR: " + ex);
            ex.printStackTrace(System.err);
        } catch (org.omg.CORBA.UserException ex) {
            System.err.println("ERROR: " + ex);
            ex.printStackTrace(System.err);
        }
    }
}
```
