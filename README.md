#include <stdio.h>
#include <stdlib.h>
#include <math.h>
#include <string.h>
#include <time.h>
#pragma warning(disable:4996)

void staffLogin();
void addStaff();
void modifyStaffInfo();
void deleteStaffInfo();
void searchStaff();
void displayStaffInfo();
void dataRecovery();
char* getPlaceOfBirthName(char* placeCode);

typedef struct {
    char birthYear[3];
    char birthMonth[3];
    char birthDay[3];
    char placeOfBirth[3];
    char gender[5];
}IC;

typedef struct{
    char staffID[7];       
    char name[50];          
    char password[20];      
    char position[50]; 
    IC ic;
    char placeBirth[50];
    char gender;
    char phone[13];
    char email[50];
}Staff;

int validateIC(IC ic) {
    time_t t = time(NULL);
    struct tm tm = *localtime(&t);
    int currentYear = tm.tm_year + 1900;

    int birthYear = (ic.birthYear[0] - '0') * 10 + (ic.birthYear[1] - '0') + 2000;

    if (birthYear < 1900 || birthYear > currentYear) {
        return 0;
    }

    int birthMonth = (ic.birthMonth[0] - '0') * 10 + (ic.birthMonth[1] - '0');

    if (birthMonth < 1 || birthMonth > 12) {
        return 0;
    }

    int birthDay = (ic.birthDay[0] - '0') * 10 + (ic.birthDay[1] - '0');

    int daysInMonth = 31;
    if (birthMonth == 4 || birthMonth == 6 || birthMonth == 9 || birthMonth == 11) {
        daysInMonth = 30;
    }
    else if (birthMonth == 2) {

        daysInMonth = (birthYear % 4 == 0 && (birthYear % 100 != 0 || birthYear % 400 == 0)) ? 29 : 28;
    }

    if (birthDay < 1 || birthDay > daysInMonth) {
        return 0;
    }

    return 1;
}

void main() {
    int choice;

    staffLogin();

    do {
        printf("\nWelcome To Staff Information Management System\n");
        printf("==============================================\n");
        printf("1. Add New Staff\n");
        printf("2. Display Staff\n");
        printf("3. Search for Staff\n");
        printf("4. Modify Staff Information\n");
        printf("5. Delete Staff\n");
        printf("6. Data Recovery\n");
        printf("7. Exit\n");
        printf("==============================================\n");

        printf("Enter your choice: ");
        scanf("%d", &choice);

        switch (choice) {
        case 1:
            system("cls");
            addStaff();
            break;

        case 2:
            system("cls");
            displayStaffInfo();
            break;

        case 3:
            system("cls");
            searchStaff();
            break;

        case 4:
            system("cls");
            modifyStaffInfo();
            break;

        case 5:
            system("cls");
            deleteStaffInfo();
            break;

        case 6:
            system("cls");
            dataRecovery();
            break;

        default:
            printf("Invalid choice. Please try again.\n");
        }
    } while (choice != 7);

    system("pause");
}

void staffLogin(){

    char username[50];
    char password[20];
    int attempts = 3;
    int loginSuccess = 0;

    while (attempts > 0 && !loginSuccess) {
        printf("\nStaff Login\n");
        printf("===========\n");

        printf("Username: "); // username: admin
        scanf(" %s", username); // password: password

        printf("Password: ");
        scanf(" %s", password);

        if (strcmp(username, "admin") == 0 && strcmp(password, "password") == 0) {
            printf("\nLogin successful!\n");
            loginSuccess = 1;
            break;
        }

        if (!loginSuccess) {
            printf("\nIncorrect username or password. Please try again.\n");
            attempts--;
            printf("Attempts left: %d\n", attempts);
        }
    }

    if (!loginSuccess) {
        printf("\nLogin failed. Exiting...\n");
        exit(-1);
    }

}

void addStaff()
{
    FILE* staffFile;
    FILE* backupFile;
    Staff staff;
    Staff tempStaff;
    char choice = 'y'; 

    staffFile = fopen("staff.bin", "ab");
    backupFile = fopen("backup.bin", "ab");

    if (!staffFile || !backupFile) {
        printf("Error opening file!\n");
        exit(-1);
    }

    do {
        printf("\n");
        printf("Add New Staff\n");
        printf("=============\n");

        printf("Enter New Staff's ID (Enter 'XXX' to menu)      > ");
        scanf(" %s", staff.staffID);

        if (strcmp(staff.staffID, "XXX") == 0) {
            break;
        }

        int idExists = 0;
        FILE* checkFile = fopen("staff.bin", "rb");

        if (checkFile) {
            while (fread(&tempStaff, sizeof(Staff), 1, checkFile) == 1) {
                if (strcmp(tempStaff.staffID, staff.staffID) == 0) {
                    printf("ID already exists. Please enter a different ID.\n");
                    idExists = 1;
                    break;
                }
            }
            fclose(checkFile);
        }

        if (idExists) {
            continue; 
        }

        printf("Enter New Staff's Name                          > ");
        scanf(" %[^\n]", staff.name);

        printf("Enter New Staff's Password                      > ");
        scanf(" %[^\n]", staff.password);

        printf("Enter New Staff's Position                      > ");
        scanf(" %[^\n]", staff.position);

        int validIC = 0;
        while (!validIC) {
            printf("Enter New Staff's IC Number (xxxxxx-xx-xxxx)    > ");
            scanf(" %2s%2s%2s-%2s-%4s", staff.ic.birthYear, staff.ic.birthMonth, staff.ic.birthDay, staff.ic.placeOfBirth, staff.ic.gender);

            validIC = validateIC(staff.ic);

            if (!validIC) {
                printf("Invalid IC format. Please enter again.\n");
            }
            else {
                int lastDigit = staff.ic.gender[3] - '0';
                staff.gender = (lastDigit % 2 == 1) ? 'M' : 'F';
                strcpy(staff.placeBirth, getPlaceOfBirthName(staff.ic.placeOfBirth));
                printf("Staff's Place of Birth                          > %s\n", staff.placeBirth);
                printf("Staff's Gender                                  > %c\n", staff.gender);

                printf("Enter New Staff's Phone Number                  > ");
                scanf(" %[^\n]", staff.phone);

                printf("Enter New Staff's Email                         > ");
                scanf(" %[^\n]", staff.email);

                fwrite(&staff, sizeof(Staff), 1, staffFile);
                fwrite(&staff, sizeof(Staff), 1, backupFile);

                printf("New staff added successfully!\n");
            }
        }
        printf("\n");

        printf("Do you want to add another staff member? (y/n): ");
        scanf(" %c", &choice);

    } while (choice == 'y' || choice == 'Y');

    fclose(staffFile);
}

void modifyStaffInfo() {
    FILE* staffFile;
    FILE* tempFile;
    FILE* backupFile;
    Staff staff;
    char targetID[10];
    int found = 0;
    char confirmModify;
    char continueModification;

    do {
        staffFile = fopen("staff.bin", "rb");
        if (!staffFile) {
            printf("Error opening staff file!\n");
            exit(-1);
        }

        tempFile = fopen("temp.bin", "wb");
        if (!tempFile) {
            printf("Error creating temporary file!\n");
            exit(-1);
        }

        backupFile = fopen("backup.bin", "wb");
        if (!backupFile) {
            printf("Error to open!\n");
            exit(-1);
        }

        system("cls");

        printf("\nCurrent Staff Details:\n");
        printf("___________________________________________________________________________________________________________\n");
        printf("|%-10s|%-20s|%-20s|%-20s|%-10s|%-20s|\n", "Staff ID", "Name", "Position", "IC", "Gender", "Place of Birth");
        printf("|----------|--------------------|--------------------|--------------------|----------|--------------------|\n");
        while (fread(&staff, sizeof(Staff), 1, staffFile) == 1) {
            printf("|%-10s|%-20s|%-20s|%2s%2s%2s-%2s-%4s      |%-10c|%-20s|\n", staff.staffID, staff.name, staff.position, staff.ic.birthYear, staff.ic.birthMonth, staff.ic.birthDay, staff.ic.placeOfBirth, staff.ic.gender, staff.gender, getPlaceOfBirthName(staff.ic.placeOfBirth));
        }
        printf("===========================================================================================================\n");
        printf("\nEnter Staff ID to modify (Enter 'XXX' back to menu): "); 
        scanf(" %s", targetID);

        if (strcmp(targetID, "XXX") == 0) {
            break;
        }

        found = 0;

        // Reset file pointer
        rewind(staffFile);

        while (fread(&staff, sizeof(Staff), 1, staffFile) == 1) {
            if (strcmp(staff.staffID, targetID) == 0) {
                found = 1;

                printf("\nStaff Information Found:\n");
                printf("__________________________\n");
                printf("Staff ID      : %s\n", staff.staffID);
                printf("Name          : %s\n", staff.name);
                printf("Position      : %s\n", staff.position);
                printf("Place of Birth: %s\n", getPlaceOfBirthName(staff.ic.placeOfBirth));
                printf("IC Number     : %s%s%s-%s-%s\n", staff.ic.birthYear, staff.ic.birthMonth, staff.ic.birthDay, staff.ic.placeOfBirth, staff.ic.gender);
                printf("Gender        : %c\n", staff.gender);
                printf("Phone         : %s\n", staff.phone);
                printf("Email         : %s\n", staff.email);

                printf("\nDo you want to modify this staff's information? (y/n): ");
                scanf(" %c", &confirmModify);

                if (confirmModify == 'y' || confirmModify == 'Y') {
                    printf("\nEnter new information :\n");
                    printf("New Name(Current Name: %s): ", staff.name);
                    scanf(" %[^\n]", staff.name);

                    printf("New Position(Current Position: %s): ", staff.position);
                    scanf(" %[^\n]", staff.position);

                    printf("Enter New IC Number (Current IC: %s%s%s-%s-%s): ", staff.ic.birthYear, staff.ic.birthMonth, staff.ic.birthDay, staff.ic.placeOfBirth, staff.ic.gender);
                    scanf(" %2s%2s%2s-%2s-%4s", staff.ic.birthYear, staff.ic.birthMonth, staff.ic.birthDay, staff.ic.placeOfBirth, staff.ic.gender);

                    int lastDigit = staff.ic.gender[3] - '0';
                    if (lastDigit % 2 == 1) {
                        staff.gender = 'M';
                    }
                    else {
                        staff.gender = 'F';
                    }

                    printf("New Phone Number (Current Phone Number: Phone: %s): ", staff.phone);
                    scanf(" %[^\n]", staff.phone);

                    printf("New Email (Current Email: %s): ", staff.email);
                    scanf(" %[^\n]", staff.email);
                }
            }
            fwrite(&staff, sizeof(Staff), 1, tempFile);
            fwrite(&staff, sizeof(Staff), 1, backupFile);
        }

        fclose(staffFile);
        fclose(tempFile);
        fclose(backupFile);

        remove("staff.bin");
        rename("temp.bin", "staff.bin");

        if (!found) {
            printf("Staff with ID %s not found.\n", targetID);
        }
        else {
            printf("\nStaff information updated successfully!\n");
        }

        printf("\nDo you want to do another modification? (y/n): ");
        scanf(" %c", &continueModification);

    } while (continueModification == 'y' || continueModification == 'Y');
}

void deleteStaffInfo() {
    FILE* staffFile;
    Staff staff;
    int choice;
    char targetID[10];
    char confirmDelete;
    int found = 0;
    int deletionCount = 0;

    staffFile = fopen("staff.bin", "rb");

    if (!staffFile) {
        printf("Error opening staff file!\n");
        return;
    }

    printf("\nCurrent Staff Details:\n");
    printf("__________________________________________________________________________________________________________\n");
    printf("|%-10s|%-20s|%-20s|%-20s|%-10s|%-20s|\n", "Staff ID", "Name", "Position", "IC", "Gender", "Place of Birth");
    printf("|----------|--------------------|--------------------|--------------------|----------|--------------------|\n");
    while (fread(&staff, sizeof(Staff), 1, staffFile) == 1) {
        printf("|%-10s|%-20s|%-20s|%2s%2s%2s-%2s-%4s      |%-10c|%-20s|\n", staff.staffID, staff.name, staff.position, staff.ic.birthYear, staff.ic.birthMonth, staff.ic.birthDay, staff.ic.placeOfBirth, staff.ic.gender, staff.gender, getPlaceOfBirthName(staff.ic.placeOfBirth));
    }
    printf("===========================================================================================================\n");

    fclose(staffFile);

    printf("\nDelete Staff\n");
    printf("=============\n");
    printf("1. Delete All Staff\n");
    printf("2. Delete Staff by ID\n");
    printf("3. Back\n");
    printf("Enter your choice >");
    scanf("%d", &choice);

    switch (choice) {
    case 1:
        printf("Are you sure you want to delete all staff? (y/n): ");
        scanf(" %c", &confirmDelete);
        if (confirmDelete == 'y' || confirmDelete == 'Y') {
            staffFile = fopen("staff.bin", "wb");
            if (!staffFile) {
                printf("Error opening staff file!\n");
                return;
            }
            fclose(staffFile);
            printf("All staff deleted successfully!\n");
            return;
        }
        else {
            printf("Deletion cancelled.\n");
            return;
        }
        break;

    case 2:
        printf("Enter Staff ID to delete: ");
        scanf(" %s", targetID);

        FILE* tempFile = fopen("temp.bin", "wb");

        if (!tempFile) {
            printf("Error creating temporary file!\n");
            return;
        }

        staffFile = fopen("staff.bin", "rb");

        if (!staffFile) {
            printf("Error opening staff file!\n");
            fclose(tempFile);
            return;
        }

        while (fread(&staff, sizeof(Staff), 1, staffFile) == 1) {
            if (strcmp(staff.staffID, targetID) == 0) {
                found = 1;
                printf("\nStaff Information Found:\n");
                printf("_________________________________________________________________\n");
                printf("|%-10s|%-20s|%-20s|%-10s|\n", "Staff ID", "Name", "Position", "Gender");
                printf("|----------|--------------------|--------------------|----------|\n");
                printf("|%-10s|%-20s|%-20s|%-10c|\n", staff.staffID, staff.name, staff.position, staff.gender);
                printf("=================================================================\n");

                printf("Do you want to delete this staff? (y/n): ");
                scanf(" %c", &confirmDelete);
                if (confirmDelete == 'y' || confirmDelete == 'Y') {
                    printf("Staff %s deleted successfully!\n", targetID);
                    deletionCount++;
                }

                else {
                    printf("Deletion cancelled.\n");
                    fwrite(&staff, sizeof(Staff), 1, tempFile);
                }
            }
            else {
                fwrite(&staff, sizeof(Staff), 1, tempFile);
            }
        }

        fclose(staffFile);
        fclose(tempFile);

        if (!found) {
            printf("Staff with ID %s not found.\n", targetID);
        }
        else if (deletionCount == 0) {
            printf("No staff deleted.\n");
        }

        remove("staff.bin");
        rename("temp.bin", "staff.bin");
        break;

    case 3:
        break;

    default:
        printf("Invalid choice.\n");
        return;
    }
}

void searchStaff() {
    FILE* staffFile;
    Staff staff;
    int choice;
    char searchName[50];
    int found = 0;

    staffFile = fopen("staff.bin", "rb");
    if (!staffFile) {
        printf("Error opening staff file!\n");
        exit(-1);
    }

    printf("\nCurrent Staff Details:\n");
    printf("___________________________________________________________________________________________________________\n");
    printf("|%-10s|%-20s|%-20s|%-20s|%-10s|%-20s|\n", "Staff ID", "Name", "Position", "IC", "Gender", "Place of Birth");
    printf("|----------|--------------------|--------------------|--------------------|----------|--------------------|\n");
    while (fread(&staff, sizeof(Staff), 1, staffFile) == 1) {
        printf("|%-10s|%-20s|%-20s|%2s%2s%2s-%2s-%4s      |%-10c|%-20s|\n", staff.staffID, staff.name, staff.position, staff.ic.birthYear, staff.ic.birthMonth, staff.ic.birthDay, staff.ic.placeOfBirth, staff.ic.gender, staff.gender, getPlaceOfBirthName(staff.ic.placeOfBirth));
    }
    printf("===========================================================================================================\n");

    fclose(staffFile);

    printf("\nSearch Staff\n");
    printf("============\n");
    printf("1. Search by ID\n");
    printf("2. Search by Name\n");
    printf("3. Back\n");
    printf("Enter your choice: ");
    scanf("%d", &choice);

    switch (choice) {
    case 1:
        staffFile = fopen("staff.bin", "rb");

        if (!staffFile) {
            printf("Error opening staff file!\n");
            exit(-1);
        }

        printf("Enter Staff ID to search: ");
        scanf(" %s", searchName);

        printf("\nSearch Results by ID:\n");
        printf("___________________________________________________________________________________________________________\n");
        printf("|%-10s|%-20s|%-20s|%-20s|%-10s|%-20s|\n", "Staff ID", "Name", "Position", "IC", "Gender", "Place of Birth");
        printf("|----------|--------------------|--------------------|--------------------|----------|--------------------|\n");
        while (fread(&staff, sizeof(Staff), 1, staffFile) == 1) {
            if (strcmp(staff.staffID, searchName) == 0) {
                printf("|%-10s|%-20s|%-20s|%2s%2s%2s-%2s-%4s      |%-10c|%-20s|\n", staff.staffID, staff.name, staff.position, staff.ic.birthYear, staff.ic.birthMonth, staff.ic.birthDay, staff.ic.placeOfBirth, staff.ic.gender, staff.gender, getPlaceOfBirthName(staff.ic.placeOfBirth));
            }
        }
        printf("===========================================================================================================\n");

        fclose(staffFile); 
        break;

    case 2:
        staffFile = fopen("staff.bin", "rb"); 
        if (!staffFile) {
            printf("Error opening staff file!\n");
            exit(-1);
        }

        printf("Enter Name to search : ");
        scanf(" %[^\n]", searchName);
        printf("\nSearch Results by Name:\n");
        printf("___________________________________________________________________________________________________________\n");
        printf("|%-10s|%-20s|%-20s|%-20s|%-10s|%-20s|\n", "Staff ID", "Name", "Position", "IC", "Gender", "Place of Birth");
        printf("|----------|--------------------|--------------------|--------------------|----------|--------------------|\n");

        found = 0;

        

        while (fread(&staff, sizeof(Staff), 1, staffFile) == 1) {
            
            if (strlen(searchName) == 1 && staff.name[0] == searchName[0]) {
                printf("|%-10s|%-20s|%-20s|%2s%2s%2s-%2s-%4s      |%-10c|%-20s|\n", staff.staffID, staff.name, staff.position, staff.ic.birthYear, staff.ic.birthMonth, staff.ic.birthDay, staff.ic.placeOfBirth, staff.ic.gender, staff.gender, getPlaceOfBirthName(staff.ic.placeOfBirth));
                found = 1;
            }
            else if (strncmp(staff.name, searchName, strlen(searchName)) == 0) {
                printf("|%-10s|%-20s|%-20s|%2s%2s%2s-%2s-%4s      |%-10c|%-20s|\n", staff.staffID, staff.name, staff.position, staff.ic.birthYear, staff.ic.birthMonth, staff.ic.birthDay, staff.ic.placeOfBirth, staff.ic.gender, staff.gender, getPlaceOfBirthName(staff.ic.placeOfBirth));
                found = 1;
            }
        }

        printf("===========================================================================================================\n");

        fclose(staffFile); 

        if (!found) {
            printf("No staff found with the entered name.\n");
        }
        break;

    case 3:
        break;

    default:
        printf("Invalid choice.\n");
        break;
    }
}

void displayStaffInfo() {
    FILE* staffFile;
    Staff staff;
    int choice;
    char targetPosition[50];
    char targetGender;
    char targetPlaceOfBirth[50];

    staffFile = fopen("staff.bin", "rb");
    if (!staffFile) {
        printf("Error opening staff file!\n");
        exit(-1);
    }

    printf("\nDisplay Staff Information:\n");
    printf("1. Display All\n");
    printf("2. Display by Position\n");
    printf("3. Display by Gender\n");
    printf("4. Display by Place of Birth\n");
    printf("5. Back\n");

    printf("Enter your choice: ");
    scanf(" %d", &choice);

    switch (choice) {
    case 1:
        printf("\nDisplaying All Staff Information:\n");
        printf("________________________________________________________________________________________________________________________________________________\n");
        printf("|%-10s|%-20s|%-20s|%-20s|%-10s|%-20s|%-15s|%-20s|\n", "Staff ID", "Name", "Position", "IC", "Gender", "Place of Birth", "Phone Number", "Email");
        printf("|----------|--------------------|--------------------|--------------------|----------|--------------------|---------------|--------------------|\n");

        while (fread(&staff, sizeof(Staff), 1, staffFile) == 1) {
            printf("|%-10s|%-20s|%-20s|%2s%2s%2s-%2s-%4s      |%-10c|%-20s|%-15s|%-20s|\n", staff.staffID, staff.name, staff.position, staff.ic.birthYear, staff.ic.birthMonth, staff.ic.birthDay, staff.ic.placeOfBirth, staff.ic.gender, staff.gender, getPlaceOfBirthName(staff.ic.placeOfBirth), staff.phone, staff.email);
        }

        printf("================================================================================================================================================\n");

        break;

    case 2:
        printf("____________________________________________________\n");
        printf("|                 List of Positions                |\n");
        printf("|--------------------------------------------------|\n");

        int positionFound = 0;
        char prevPosition[50] = "";

        while (fread(&staff, sizeof(Staff), 1, staffFile) == 1) {
            if (strcmp(staff.position, prevPosition) != 0) { 
                printf("|%-50s|\n", staff.position);
                strcpy(prevPosition, staff.position);
                positionFound = 1;
            }
        }

        printf("====================================================\n");

        if (!positionFound) {
            printf("No positions found.\n");
            fclose(staffFile);
            break;
        }

        printf("\nEnter the Position to Display: ");
        scanf(" %[^\n]", targetPosition);
        printf("\nDisplaying Staff with Position %s:\n", targetPosition);
        printf("________________________________________________________________________________________________________________________________________________\n");
        printf("|%-10s|%-20s|%-20s|%-20s|%-10s|%-20s|%-15s|%-20s|\n", "Staff ID", "Name", "Position", "IC", "Gender", "Place of Birth", "Phone Number", "Email");
        printf("|----------|--------------------|--------------------|--------------------|----------|--------------------|---------------|--------------------|\n");

        rewind(staffFile);
        while (fread(&staff, sizeof(Staff), 1, staffFile) == 1) {
            if (strcmp(staff.position, targetPosition) == 0) {
                printf("|%-10s|%-20s|%-20s|%2s%2s%2s-%2s-%4s      |%-10c|%-20s|%-15s|%-20s|\n", staff.staffID, staff.name, staff.position, staff.ic.birthYear, staff.ic.birthMonth, staff.ic.birthDay, staff.ic.placeOfBirth, staff.ic.gender, staff.gender, getPlaceOfBirthName(staff.ic.placeOfBirth), staff.phone, staff.email);
            }
        }

        printf("================================================================================================================================================\n");

        break;

    case 3:
        printf("\nEnter the Gender to Display (M/F): ");
        scanf(" %c", &targetGender);
        printf("\nDisplaying Staff with Gender %c:\n", targetGender);
        printf("________________________________________________________________________________________________________________________________________________\n");
        printf("|%-10s|%-20s|%-20s|%-20s|%-10s|%-20s|%-15s|%-20s|\n", "Staff ID", "Name", "Position", "IC", "Gender", "Place of Birth", "Phone Number", "Email");
        printf("|----------|--------------------|--------------------|--------------------|----------|--------------------|---------------|--------------------|\n");

        while (fread(&staff, sizeof(Staff), 1, staffFile) == 1) {
            if (staff.gender == targetGender) {
                printf("|%-10s|%-20s|%-20s|%2s%2s%2s-%2s-%4s      |%-10c|%-20s|%-15s|%-20s|\n", staff.staffID, staff.name, staff.position, staff.ic.birthYear, staff.ic.birthMonth, staff.ic.birthDay, staff.ic.placeOfBirth, staff.ic.gender, staff.gender, getPlaceOfBirthName(staff.ic.placeOfBirth), staff.phone, staff.email);
            }
        }

        printf("================================================================================================================================================\n");

        break;

    case 4:
        printf("_________________\n");
        printf("|List of Place  |\n");
        printf("|---------------|\n");
        printf("|Johor          |\n");
        printf("|Kedah          |\n");
        printf("|Kelantan       |\n");
        printf("|Malacca        |\n");
        printf("|Negeri Sembilan|\n");
        printf("|Pahang         |\n");
        printf("|Penang         |\n");
        printf("|Perak          |\n");
        printf("|Perlis         |\n");
        printf("|Selangor       |\n");
        printf("|Terengganu     |\n");
        printf("|Sabah          |\n");
        printf("|Sarawak        |\n");
        printf("|Kuala Lumpur   |\n");
        printf("=================\n");

        printf("\nEnter the Place of Birth to Display: ");
        scanf(" %[^\n]", targetPlaceOfBirth); 
        printf("\nDisplaying Staff with Place of Birth %s:\n", targetPlaceOfBirth);
        printf("_______________________________________________________________________________________________________________________________________________\n");
        printf("|%-10s|%-20s|%-20s|%-20s|%-10s|%-20s|%-15s|%-20s|\n", "Staff ID", "Name", "Position", "IC", "Gender", "Place of Birth", "Phone Number", "Email");
        printf("|----------|--------------------|--------------------|--------------------|----------|--------------------|---------------|--------------------|\n");

        while (fread(&staff, sizeof(Staff), 1, staffFile) == 1) {
            if (strcmp(staff.placeBirth, targetPlaceOfBirth) == 0) {
                printf("|%-10s|%-20s|%-20s|%2s%2s%2s-%2s-%4s      |%-10c|%-20s|%-15s|%-20s|\n", staff.staffID, staff.name, staff.position, staff.ic.birthYear, staff.ic.birthMonth, staff.ic.birthDay, staff.ic.placeOfBirth, staff.ic.gender, staff.gender, getPlaceOfBirthName(staff.ic.placeOfBirth), staff.phone, staff.email);
            }
        }

        printf("================================================================================================================================================\n");

        break;

    case 5:
        break;

    default:
        printf("Invalid choice.\n");
        break;
    }

    fclose(staffFile);
}

void dataRecovery() {
    FILE* backupFile;
    Staff staff;

    backupFile = fopen("backup.bin", "rb");
    if (!backupFile) {
        printf("Backup file not found!\n");
        return;
    }

    printf("\nBackup data found. Displaying backup staff information:\n");
    printf("=========================================================\n");
    while (fread(&staff, sizeof(Staff), 1, backupFile) == 1) {
        printf("|Staff ID: %s\n", staff.staffID);
        printf("|Name    : %s\n", staff.name);
        printf("|IC      : %s%s%s-%s-%s\n", staff.ic.birthYear, staff.ic.birthMonth, staff.ic.birthDay, staff.ic.placeOfBirth, staff.ic.gender);
        printf("|Position: %s\n", staff.position);
        printf("|Phone   : %s\n", staff.phone);
        printf("|Email   : %s\n", staff.email);
        printf("=========================================================\n");
    }

    fclose(backupFile);

    char choice;
    printf("\nDo you want to recover the data? (y/n): ");
    scanf(" %c", &choice);

    if (choice == 'y' || choice == 'Y') {
        FILE* staffFile = fopen("staff.bin", "wb");
        if (!staffFile) {
            printf("Error opening staff file!\n");
            exit(-1);
        }

        backupFile = fopen("backup.bin", "rb");
        if (!backupFile) {
            printf("Error opening backup file!\n");
            fclose(staffFile);
            exit(-1);
        }

        while (fread(&staff, sizeof(Staff), 1, backupFile) == 1) {
            fwrite(&staff, sizeof(Staff), 1, staffFile);
        }

        fclose(staffFile);
        fclose(backupFile);

        printf("Data recovery successful!\n");
    }
    else {
        printf("Data recovery cancelled.\n");
    }
}

char* getPlaceOfBirthName(char* placeCode) 
{
    if (strcmp(placeCode, "01") == 0) return "Johor";
    else if (strcmp(placeCode, "02") == 0) return "Kedah";
    else if (strcmp(placeCode, "03") == 0) return "Kelantan";
    else if (strcmp(placeCode, "04") == 0) return "Malacca";
    else if (strcmp(placeCode, "05") == 0) return "Negeri Sembilan";
    else if (strcmp(placeCode, "06") == 0) return "Pahang";
    else if (strcmp(placeCode, "07") == 0) return "Penang";
    else if (strcmp(placeCode, "08") == 0) return "Perak";
    else if (strcmp(placeCode, "09") == 0) return "Perlis";
    else if (strcmp(placeCode, "10") == 0) return "Selangor";
    else if (strcmp(placeCode, "11") == 0) return "Terengganu";
    else if (strcmp(placeCode, "12") == 0) return "Sabah";
    else if (strcmp(placeCode, "13") == 0) return "Sarawak";
    else if (strcmp(placeCode, "14") == 0) return "Kuala Lumpur";
    else return "Unknown";
}

