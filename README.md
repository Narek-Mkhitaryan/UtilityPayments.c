#include <stdio.h>
#include <stdlib.h>

#define MAX_MONTHS 100 // Maximum number of months to store data
#define NUM_TYPES 3    // Number of types of utility services
#define NUM_LATEST_PAYMENTS 3 // Number of latest payments to average

// Structure to store utility payments for each month
struct UtilityPayment {
    int month;
    float water;
    float gas;
    float electricity;
};

// Function to save utility payment data to a file
void saveUtilityPayment(struct UtilityPayment *payment) {
    FILE *file = fopen("utility_payments.txt", "a");
    if (file == NULL) {
        printf("Error opening file.\n");
        exit(1);
    }

    fprintf(file, "%d %.2f %.2f %.2f\n", payment->month, payment->water, payment->gas, payment->electricity);
    fclose(file);
}

// Function to calculate the average cost of a specific utility type
float calculateAverageCost(struct UtilityPayment *payments, int count, int utilityType) {
    float total = 0;
    int start = count >= NUM_TYPES ? count - NUM_TYPES : 0; // Start calculating average from three months ago
    for (int i = start; i < count; ++i) {
        if (utilityType == 1) // Water
            total += payments[i].water;
        else if (utilityType == 2) // Gas
            total += payments[i].gas;
        else if (utilityType == 3) // Electricity
            total += payments[i].electricity;
    }
    return total / (count - start);
}

// Function to print if the latest utility payment exceeds or is less than the average cost
void printExceededUtilityType(float latestValue, float averageValue, const char *utilityType) {
    float difference = latestValue - averageValue;
    if (difference > 0) {
        printf("The latest %s payment exceeds the average %s cost by %.2f$.\n", utilityType, utilityType, difference);
    } else if (difference < 0) {
        printf("The latest %s payment is less than the average %s cost by %.2f$.\n", utilityType, utilityType, -difference);
    } else {
        printf("The latest %s payment matches the average %s cost.\n", utilityType, utilityType);
    }
}

// Function to print the details of the latest utility payments
void printLatestPayments(struct UtilityPayment *payments, int count) {
    printf("Latest payments:\n");
    int start = count - NUM_LATEST_PAYMENTS >= 0 ? count - NUM_LATEST_PAYMENTS : 0;
    for (int i = start; i < count; ++i) {
        printf("Month: %d | Water: %.2f | Gas: %.2f | Electricity: %.2f\n", payments[i].month,
               payments[i].water, payments[i].gas, payments[i].electricity);
    }
}

int main() {
    struct UtilityPayment payments[MAX_MONTHS] = {0};
    int count = 0;

    // Check if the file exists and create it if it doesn't
    FILE *file = fopen("utility_payments.txt", "a+");
    if (file == NULL) {
        printf("Error opening or creating file.\n");
        exit(1);
    }
    fclose(file); // Close the file after creation (it will be opened again in the saveUtilityPayment function)

    // Load data from the file if it exists
    file = fopen("utility_payments.txt", "r");
    if (file != NULL) {
        while (fscanf(file, "%d %f %f %f", &payments[count].month, &payments[count].water,
                      &payments[count].gas, &payments[count].electricity) != EOF) {
            count++;
        }
        fclose(file);
    } else {
        printf("Error opening file.\n");
        exit(1);
    }

    int latestMonth = (count % MAX_MONTHS) + 1;
    printf("Enter the latest payment for Water, Gas, and Electricity (separated by spaces, value is $): ");
    scanf("%f %f %f", &payments[latestMonth - 1].water, &payments[latestMonth - 1].gas, &payments[latestMonth - 1].electricity);
    payments[latestMonth - 1].month = latestMonth;
    saveUtilityPayment(&payments[latestMonth - 1]);

    float averageWaterCost = calculateAverageCost(payments, count, 1);
    float averageGasCost = calculateAverageCost(payments, count, 2);
    float averageElectricityCost = calculateAverageCost(payments, count, 3);

    float latestWater = payments[latestMonth - 1].water;
    float latestGas = payments[latestMonth - 1].gas;
    float latestElectricity = payments[latestMonth - 1].electricity;

    float totalLatest = latestWater + latestGas + latestElectricity; // Total amount of the new payment

    printf("\nSummary:\n");
    printf("------------\n");
    printf("Total latest payments: %.2f$\n", totalLatest);
    printf("------------\n");
    printExceededUtilityType(latestWater, averageWaterCost, "Water");
    printf("\n");
    printExceededUtilityType(latestGas, averageGasCost, "Gas");
    printf("\n");
    printExceededUtilityType(latestElectricity, averageElectricityCost, "Electricity");
    printf("\n");
    printf("\n");
    printLatestPayments(payments, count);

    return 0;
}
