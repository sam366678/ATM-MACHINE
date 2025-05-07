#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define FILENAME "users.dat"

typedef struct {
    char id[20];
    char name[50];
    char pin[10];
    float balance;
} User;

// Simple Caesar cipher for encryption/decryption
void encrypt(char *str) {
    while (*str) {
        *str = *str + 3;
        str++;
    }
}

void decrypt(char *str) {
    while (*str) {
        *str = *str - 3;
        str++;
    }
}

// Save user to file
void saveUser(User user) {
    FILE *fp = fopen(FILENAME, "ab");
    if (!fp) {
        printf("Error opening file!\n");
        return;
    }
    fwrite(&user, sizeof(User), 1, fp);
    fclose(fp);
}

// Search for a user
int findUser(char *id, User *foundUser) {
    FILE *fp = fopen(FILENAME, "rb");
    if (!fp) return 0;

    User temp;
    while (fread(&temp, sizeof(User), 1, fp)) {
        decrypt(temp.id);
        if (strcmp(temp.id, id) == 0) {
            *foundUser = temp;
            fclose(fp);
            return 1;
        }
        encrypt(temp.id); // Restore after comparison
    }
    fclose(fp);
    return 0;
}

// Update user in file
void updateUser(User updatedUser) {
    FILE *fp = fopen(FILENAME, "rb+");
    if (!fp) return;

    User temp;
    while (fread(&temp, sizeof(User), 1, fp)) {
        decrypt(temp.id);
        if (strcmp(temp.id, updatedUser.id) == 0) {
            fseek(fp, -sizeof(User), SEEK_CUR);
            encrypt(updatedUser.id);
            encrypt(updatedUser.pin);
            fwrite(&updatedUser, sizeof(User), 1, fp);
            fclose(fp);
            return;
        }
        encrypt(temp.id);
    }
    fclose(fp);
}

// Registration
void registerUser() {
    User user;
    printf("Enter Name: ");
    scanf(" %[^\n]", user.name);
    printf("Enter ID: ");
    scanf("%s", user.id);
    printf("Enter PIN: ");
    scanf("%s", user.pin);
    user.balance = 0.0;

    encrypt(user.id);
    encrypt(user.pin);

    saveUser(user);
    printf("Registration successful!\n");
}

// Login
int login(User *user) {
    char id[20], pin[10];
    printf("Enter ID: ");
    scanf("%s", id);
    printf("Enter PIN: ");
    scanf("%s", pin);

    User temp;
    if (findUser(id, &temp)) {
        decrypt(temp.pin);
        if (strcmp(temp.pin, pin) == 0) {
            *user = temp;
            return 1;
        }
    }
    return 0;
}

// Menu Operations
void showMenu(User user) {
    int choice;
    float amount;
    char targetID[20];
    User target;

    do {
        printf("\n1. Check Balance\n2. Deposit\n3. Withdraw\n4. Transfer\n5. Change PIN\n6. Exit\n");
        printf("Enter choice: ");
        scanf("%d", &choice);

        switch (choice) {
            case 1:
                printf("Your balance is: %.2f\n", user.balance);
                break;
            case 2:
                printf("Enter amount to deposit: ");
                scanf("%f", &amount);
                user.balance += amount;
                printf("Deposit successful. New balance: %.2f\n", user.balance);
                break;
            case 3:
                printf("Enter amount to withdraw: ");
                scanf("%f", &amount);
                if (amount <= user.balance) {
                    user.balance -= amount;
                    printf("Withdrawal successful. New balance: %.2f\n", user.balance);
                } else {
                    printf("Insufficient funds!\n");
                }
                break;
            case 4:
                printf("Enter target ID: ");
                scanf("%s", targetID);
                if (findUser(targetID, &target)) {
                    printf("Enter amount to transfer: ");
                    scanf("%f", &amount);
                    if (amount <= user.balance) {
                        user.balance -= amount;
                        target.balance += amount;
                        updateUser(target);
                        printf("Transfer successful.\n");
                    } else {
                        printf("Insufficient balance!\n");
                    }
                } else {
                    printf("Target user not found.\n");
                }
                break;
            case 5:
                printf("Enter new PIN: ");
                scanf("%s", user.pin);
                printf("PIN changed successfully.\n");
                break;
            case 6:
                updateUser(user);
                printf("Goodbye!\n");
                return;
            default:
                printf("Invalid option!\n");
        }
    } while (1);
}

// Main Function
int main() {
    int option;
    User currentUser;

    while (1) {
        printf("\n--- ATM Menu ---\n");
        printf("1. Register\n2. Login\n3. Exit\n");
        printf("Enter choice: ");
        scanf("%d", &option);

        if (option == 1) {
            registerUser();
        } else if (option == 2) {
            if (login(&currentUser)) {
                printf("Login successful!\n");
                showMenu(currentUser);
            } else {
                printf("Invalid ID or PIN.\n");
            }
        } else if (option == 3) {
            printf("Exiting...\n");
            break;
        } else {
            printf("Invalid choice.\n");
        }
    }

    return 0;
}
