import java.io.*;
import java.util.Scanner;

// Room class and its subclasses
class Room {
    int roomNumber;
    boolean occupied;
    String guestName;
    int roomType; // 1 for Single, 2 for Double, 3 for Suite

    public Room(int roomNumber, int roomType) {
        this.roomNumber = roomNumber;
        this.roomType = roomType;
        this.occupied = false;
        this.guestName = "";
    }
}

class SingleRoom extends Room {
    public SingleRoom(int roomNumber) {
        super(roomNumber, 1);
    }
}

class DoubleRoom extends Room {
    public DoubleRoom(int roomNumber) {
        super(roomNumber, 2);
    }
}

class SuiteRoom extends Room {
    public SuiteRoom(int roomNumber) {
        super(roomNumber, 3);
    }
}

// Admin class
class Admin {
    private static final String ADMIN_USERNAME = "sarmad";
    private static final String ADMIN_PASSWORD = "321";

    public static void login() {
        Scanner scanner = new Scanner(System.in);
        System.out.print("Enter username: ");
        String username = scanner.nextLine();
        System.out.print("Enter password: ");
        String password = scanner.nextLine();

        if (username.equals(ADMIN_USERNAME) && password.equals(ADMIN_PASSWORD)) {
            System.out.println("Admin login successful!");
            showAdminMenu();
        } else {
            System.out.println("Admin login failed. Access denied.");
        }
    }

    public static void showAdminMenu() {
        Scanner scanner = new Scanner(System.in);
        int choice;

        while (true) {
            System.out.println("\nAdmin Menu");
            System.out.println("1. Add Room");
            System.out.println("2. Check In");
            System.out.println("3. Check Out");
            System.out.println("4. Display Rooms");
            System.out.println("5. Logout");
            System.out.print("Enter your choice: ");
            choice = scanner.nextInt();

            switch (choice) {
                case 1:
                    Reservation.addRoom();
                    break;
                case 2:
                    Reservation.checkIn();
                    break;
                case 3:
                    Reservation.checkOut();
                    break;
                case 4:
                    Reservation.displayRooms();
                    break;
                case 5:
                    System.out.println("Admin logged out.");
                    return;
                default:
                    System.out.println("Invalid choice. Please try again.");
            }
        }
    }
}

// Reservation class
public class Reservation {
    private static final int MAX_ROOMS = 100;
    private static final Room[] hotelRooms = new Room[MAX_ROOMS];
    private static int numRooms = 0;
    private static final String FILE_NAME = "C:\\ACP\\rooms.txt"; // File path for the rooms file

    public static void loadRooms() {
        File file = new File(FILE_NAME);
        if (!file.exists()) {
            System.out.println("The file " + FILE_NAME + " does not exist. Please create it.");
            return; // Exit the method if the file is not found
        }

        try (BufferedReader br = new BufferedReader(new FileReader(file))) {
            String line;
            while ((line = br.readLine()) != null) {
                String[] roomData = line.split(",");
                if (roomData.length != 4) {
                    System.out.println("Invalid room data: " + line);
                    continue; // Skip this line if it doesn't have exactly 4 elements
                }
                int roomNumber = Integer.parseInt(roomData[0]);
                int roomType = Integer.parseInt(roomData[1]);
                Room newRoom = null;

                switch (roomType) {
                    case 1:
                        newRoom = new SingleRoom(roomNumber);
                        break;
                    case 2:
                        newRoom = new DoubleRoom(roomNumber);
                        break;
                    case 3:
                        newRoom = new SuiteRoom(roomNumber);
                        break;
                    default:
                        System.out.println("Invalid room type for room number: " + roomNumber);
                        continue; // Skip this line if the room type is invalid
                }

                if (newRoom != null) {
                    newRoom.occupied = Boolean.parseBoolean(roomData[2]);
                    newRoom.guestName = roomData[3];
                    hotelRooms[numRooms++] = newRoom;
                }
            }
        } catch (IOException e) {
            System.out.println("Error loading rooms: " + e.getMessage());
        }
    }

    public static void saveRooms() {
        try (BufferedWriter bw = new BufferedWriter(new FileWriter(FILE_NAME))) {
            for (int i = 0; i < numRooms; i++) {
                Room room = hotelRooms[i];
                bw.write(room.roomNumber + "," + room.roomType + "," + room.occupied + "," + room.guestName);
                bw.newLine();
            }
        } catch (IOException e) {
            System.out.println("Error saving rooms: " + e.getMessage());
        }
    }

    public static void addRoom() {
        Scanner scanner = new Scanner(System.in);
        System.out.print("Enter room number: ");
        int roomNumber = scanner.nextInt();
        System.out.print("Enter room type (1 for Single, 2 for Double, 3 for Suite): ");
        int roomType = scanner.nextInt();

        Room newRoom = null;
        switch (roomType) {
            case 1:
                newRoom = new SingleRoom(roomNumber);
                break;
            case 2:
                newRoom = new DoubleRoom(roomNumber);
                break;
            case 3:
                newRoom = new SuiteRoom(roomNumber);
                break;
            default:
                System.out.println("Invalid room type.");
                return;
        }

        hotelRooms[numRooms++] = newRoom;
        System.out.println("Room added successfully.");
        saveRooms(); // Save the updated room list to the file
    }

    public static void checkIn() {
        Scanner scanner = new Scanner(System.in);
        System.out.print("Enter room number for check-in: ");
        int roomNumber = scanner.nextInt();

        for (int i = 0; i < numRooms; i++) {
            if (hotelRooms[i].roomNumber == roomNumber) {
                if (!hotelRooms[i].occupied) {
                    System.out.print("Enter guest name: ");
                    hotelRooms[i].guestName = scanner.next();
                    hotelRooms[i].occupied = true;
                    System.out.println("Check-in successful.");
                    saveRooms(); // Save the updated room list to the file
                    return;
                } else {
                    System.out.println("Room is already occupied.");
                    return;
                }
            }
        }
        System.out.println("Room not found.");
    }

    public static void checkOut() {
        Scanner scanner = new Scanner(System.in);
        System.out.print("Enter room number for check-out: ");
        int roomNumber = scanner.nextInt();

        for (int i = 0; i < numRooms; i++) {
            if (hotelRooms[i].roomNumber == roomNumber) {
                if (hotelRooms[i].occupied) {
                    hotelRooms[i].occupied = false;
                    hotelRooms[i].guestName = "";
                    System.out.println("Check-out successful.");
                    saveRooms(); // Save the updated room list to the file
                    return;
                } else {
                    System.out.println("Room is already vacant.");
                    return;
                }
            }
        }
        System.out.println("Room not found.");
    }

    public static void displayRooms() {
        System.out.println("Room Number | Room Type | Occupied | Guest Name");
        for (int i = 0; i < numRooms; i++) {
            Room room = hotelRooms[i];
            String roomType = (room.roomType == 1) ? "Single" : (room.roomType == 2) ? "Double" : "Suite";
            System.out.printf("%-12d | %-9s | %-8s | %s%n", room.roomNumber, roomType, room.occupied, room.guestName);
        }
    }

    public static void main(String[] args) {
        loadRooms(); // Load rooms from the file at the start
        Admin.login(); // Start the admin login process
    }
}