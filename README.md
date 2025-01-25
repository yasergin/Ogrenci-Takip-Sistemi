#include <stdio.h>
#include <string.h>
#include <stdlib.h>

#define MAX_STUDENTS 50
#define MAX_NAME_LEN 50
#define MAX_NOTE_LEN 100
#define MAX_GAINS 5
#define MAX_CLASS_NAME_LEN 50

// Öğrenci yapısı
typedef struct {
    char name[MAX_NAME_LEN];
    char surname[MAX_NAME_LEN];
    char parentName[MAX_NAME_LEN];
    char parentSurname[MAX_NAME_LEN];
    char parentPhone[15];
    int gains[MAX_GAINS];
    char note[MAX_NOTE_LEN];
    int isProcessed;
} Student;

Student students[MAX_STUDENTS];
int studentCount = 0;
char className[MAX_CLASS_NAME_LEN]; // Sınıf adı

// Kazanımlar
const char *gainsList[MAX_GAINS] = {
    "Problem durumunun ne olduğunu söyler",
    "Probleme çözüm önerisi getirir",
    "Getirdiği çözüm önerilerinden en uygun olanını yapar",
    "Ekip çalışmasına katılır",
    "Yardım gerektiğinde öğretmenine danışır"
};

// Fonksiyonlar
void saveToFile() {
    FILE *file = fopen("students.txt", "w");
    if (!file) {
        printf("Dosya kaydedilirken bir hata oluştu!\n");
        return;
    }

    fprintf(file, "Sınıf: %s\n\n", className);

    for (int i = 0; i < studentCount; i++) {
        fprintf(file, "%s %s, Veli: %s %s (%s)\n",
                students[i].name, students[i].surname,
                students[i].parentName, students[i].parentSurname,
                students[i].parentPhone);
        fprintf(file, "Kazanımlar:\n");
        for (int j = 0; j < MAX_GAINS; j++) {
            fprintf(file, "- %s: %d\n", gainsList[j], students[i].gains[j]);
        }
        fprintf(file, "Not: %s\n\n", students[i].note);
    }
    fclose(file);
    printf("Tüm bilgiler başarıyla dosyaya kaydedildi!\n");
}

void processStudent(int index) {
    printf("%s %s öğrencisi için kazanımları girin:\n", students[index].name, students[index].surname);

    // Daha önce işlenmiş mi kontrol et
    if (students[index].isProcessed) {
        printf("Bu öğrenci için daha önce kazanımlar girildi:\n");
        for (int i = 0; i < MAX_GAINS; i++) {
            printf("- %s: %d\n", gainsList[i], students[index].gains[i]);
        }
        printf("Not: %s\n", students[index].note);
    }

    // Kazanımları gir
    for (int i = 0; i < MAX_GAINS; i++) {
        int gain;
        do {
            printf("%s (0-3): ", gainsList[i]);
            if (scanf("%d", &gain) != 1) {
                printf("Geçerli bir sayı girin!\n");
                while (getchar() != '\n'); // Geçersiz girişleri temizle
                continue;
            }
            if (gain < 0 || gain > 3) {
                printf("Lütfen 0 ile 3 arasında bir değer girin.\n");
            }
        } while (gain < 0 || gain > 3);
        students[index].gains[i] = gain;
    }

    // Not gir
    printf("Bu öğrenci için bir not bırakın: ");
    getchar(); // Yeni satır karakterini temizle
    fgets(students[index].note, MAX_NOTE_LEN, stdin);
    strtok(students[index].note, "\n"); // Yeni satır karakterini kaldır

    students[index].isProcessed = 1;

    // Geri bildirim
    int zeroCount = 0, oneCount = 0, twoCount = 0;
    for (int i = 0; i < MAX_GAINS; i++) {
        if (students[index].gains[i] == 0) zeroCount++;
        if (students[index].gains[i] == 1) oneCount++;
        if (students[index].gains[i] == 2) twoCount++;
    }

    if (zeroCount > 0) {
        printf("Bu öğrencinin velisiyle iletişime geçmelisiniz.\n");
    } else if (oneCount >= 3) {
        printf("Bu öğrencinin velisiyle iletişime geçmelisiniz.\n");
    } else if (oneCount == 2) {
        printf("Bu öğrenciyi gözlemlemeye devam etmelisiniz.\n");
    } else if (twoCount > 0) {
        printf("Bu öğrenciyle bu durumu geliştirmesi için çalışmalısınız.\n");
    } else {
        printf("Bu öğrenci oldukça başarılı!\n");
    }
}

void startProcessing() {
    char choice[4];
    int processedCount = 0;

    while (processedCount < studentCount) {
        printf("Hangi öğrenci için işlem yapmak istiyorsunuz? İsim girin: ");
        char name[MAX_NAME_LEN];
        scanf("%s", name);

        int found = -1;
        for (int i = 0; i < studentCount; i++) {
            if (strcmp(students[i].name, name) == 0) {
                found = i;
                break;
            }
        }

        if (found == -1) {
            printf("Bu isimde bir öğrenci bulunamadı!\n");
        } else {
            if (!students[found].isProcessed) {
                processStudent(found);
                processedCount++;
            } else {
                printf("Bu öğrenci zaten işlenmiş.\n");
            }
        }

        if (processedCount < studentCount) {
            printf("Başka bir öğrenci için işlem yapmak istiyor musunuz? (Evet/Hayır): ");
            scanf("%s", choice);
            if (strcmp(choice, "Hayır") == 0 || strcmp(choice, "hayır") == 0) break;
        }
    }

    saveToFile();
}

int main() {
    printf("Sınıfın ismini girin: ");
    fgets(className, MAX_CLASS_NAME_LEN, stdin);
    strtok(className, "\n"); // Yeni satır karakterini kaldır

    do {
        printf("Kaç öğrenciniz var? ");
        if (scanf("%d", &studentCount) != 1 || studentCount <= 0 || studentCount > MAX_STUDENTS) {
            printf("Lütfen 1 ile %d arasında geçerli bir sayı girin.\n", MAX_STUDENTS);
            while (getchar() != '\n'); // Geçersiz girişleri temizle
            studentCount = 0;
        }
    } while (studentCount <= 0 || studentCount > MAX_STUDENTS);

    for (int i = 0; i < studentCount; i++) {
        printf("%d. öğrencinin adını girin: ", i + 1);
        scanf("%s", students[i].name);
        printf("%d. öğrencinin soyadını girin: ", i + 1);
        scanf("%s", students[i].surname);
        printf("%d. öğrencinin velisinin adını girin: ", i + 1);
        scanf("%s", students[i].parentName);
        printf("%d. öğrencinin velisinin soyadını girin: ", i + 1);
        scanf("%s", students[i].parentSurname);
        printf("%d. öğrencinin velisinin telefon numarasını girin: ", i + 1);
        scanf("%s", students[i].parentPhone);
        students[i].isProcessed = 0;
    }

    startProcessing();

    return 0;
}
