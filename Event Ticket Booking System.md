#include <stdio.h>
#include <stdlib.h>
#include <string.h>

typedef struct Event {
    int eventId;
    char name[50];
    struct Event* next;
} Event;

typedef struct QueueNode {
    char customerName[50];
    int eventId;
    struct QueueNode* next;
} QueueNode;

typedef struct Queue {
    QueueNode *front, *rear;
} Queue;

typedef struct Booking {
    int bookingId;
    char customerName[50];
    int eventId;
    struct Booking *left, *right;
} Booking;

Event* eventHead = NULL;
Queue bookingQueue = {NULL, NULL};
Booking* bookingRoot = NULL;
int bookingCounter = 1;

void addEvent(int id, const char* name) {
    Event* newEvent = (Event*)malloc(sizeof(Event));
    newEvent->eventId = id;
    strcpy(newEvent->name, name);
    newEvent->next = eventHead;
    eventHead = newEvent;
}

void displayEvents() {
    Event* temp = eventHead;
    printf("\nAvailable Events:\n");
    while (temp) {
        printf("Event ID: %d, Name: %s\n", temp->eventId, temp->name);
        temp = temp->next;
    }
}

void enqueue(const char* name, int eventId) {
    QueueNode* newNode = (QueueNode*)malloc(sizeof(QueueNode));
    strcpy(newNode->customerName, name);
    newNode->eventId = eventId;
    newNode->next = NULL;
    if (bookingQueue.rear == NULL) {
        bookingQueue.front = bookingQueue.rear = newNode;
    } else {
        bookingQueue.rear->next = newNode;
        bookingQueue.rear = newNode;
    }
    printf("Booking request added to queue.\n");
}

void processBooking() {
    if (bookingQueue.front == NULL) {
        printf("No bookings in queue.\n");
        return;
    }
    QueueNode* temp = bookingQueue.front;
    bookingQueue.front = bookingQueue.front->next;
    if (bookingQueue.front == NULL)
        bookingQueue.rear = NULL;

    Booking* newBooking = (Booking*)malloc(sizeof(Booking));
    newBooking->bookingId = bookingCounter++;
    strcpy(newBooking->customerName, temp->customerName);
    newBooking->eventId = temp->eventId;
    newBooking->left = newBooking->right = NULL;

    Booking** curr = &bookingRoot;
    while (*curr != NULL) {
        if (newBooking->bookingId < (*curr)->bookingId)
            curr = &((*curr)->left);
        else
            curr = &((*curr)->right);
    }
    *curr = newBooking;

    printf("Booking processed for %s (Event ID: %d, Booking ID: %d)\n", 
           newBooking->customerName, newBooking->eventId, newBooking->bookingId);
    free(temp);
}

void displayBookings(Booking* root) {
    if (!root) return;
    displayBookings(root->left);
    printf("Booking ID: %d | Customer: %s | Event ID: %d\n", root->bookingId, root->customerName, root->eventId);
    displayBookings(root->right);
}

void menu() {
    int choice, eventId;
    char name[50];
    while (1) {
        printf("\n--- Event Ticket Booking System ---\n");
        printf("1. Add Event\n");
        printf("2. View Events\n");
        printf("3. Book Ticket (Queue)\n");
        printf("4. Process Booking\n");
        printf("5. View All Bookings\n");
        printf("6. Exit\n");
        printf("Enter your choice: ");
        scanf("%d", &choice);
        getchar();  // Consume newline

        switch (choice) {
            case 1:
                printf("Enter Event ID: ");
                scanf("%d", &eventId);
                getchar();
                printf("Enter Event Name: ");
                fgets(name, sizeof(name), stdin);
                name[strcspn(name, "\n")] = 0;
                addEvent(eventId, name);
                break;

            case 2:
                displayEvents();
                break;

            case 3:
                printf("Enter Customer Name: ");
                fgets(name, sizeof(name), stdin);
                name[strcspn(name, "\n")] = 0;
                printf("Enter Event ID to Book: ");
                scanf("%d", &eventId);
                enqueue(name, eventId);
                break;

            case 4:
                processBooking();
                break;

            case 5:
                printf("\nAll Bookings:\n");
                displayBookings(bookingRoot);
                break;

            case 6:
                printf("Exiting...\n");
                exit(0);
        }
    }
}

int main() {
    menu();
    return 0;
}
