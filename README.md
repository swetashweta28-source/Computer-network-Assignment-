# Computer-network-Assignment-3
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>
#include <ctype.h>

#define PORT 8081
#define BUFFER_SIZE 1024

void to_upper(char *str)
{
    while (*str)
    {
        *str = toupper(*str);
        str++;
    }
}

void to_lower(char *str)
{
    while (*str)
    {
        *str = tolower(*str);
        str++;
    }
}

void reverse(char *str)
{
    int i = 0;
    int j = strlen(str) - 1;
    char temp;

    while (i < j)
    {
        temp = str[i];
        str[i] = str[j];
        str[j] = temp;
        i++;
        j--;
    }
}

int main()
{
    int server_fd, new_socket;
    struct sockaddr_in address;
    int addrlen = sizeof(address);

    char buffer[BUFFER_SIZE];
    char command[20], text[BUFFER_SIZE];

    server_fd = socket(AF_INET, SOCK_STREAM, 0);

    if (server_fd < 0)
    {
        perror("Socket failed");
        exit(EXIT_FAILURE);
    }

    address.sin_family = AF_INET;
    address.sin_addr.s_addr = INADDR_ANY;
    address.sin_port = htons(PORT);

    bind(server_fd, (struct sockaddr *)&address, sizeof(address));

    listen(server_fd, 5);

    printf("Server listening on port %d...\n", PORT);

    new_socket = accept(server_fd,
                        (struct sockaddr *)&address,
                        (socklen_t *)&addrlen);

    printf("Client connected.\n");

    while (1)
    {
        memset(buffer, 0, BUFFER_SIZE);

        int valread = read(new_socket, buffer, BUFFER_SIZE);

        if (valread <= 0)
            break;

        buffer[strcspn(buffer, "\n")] = 0;

        if (strcmp(buffer, "bye") == 0)
        {
            printf("Client disconnected.\n");
            break;
        }

        memset(command, 0, sizeof(command));
        memset(text, 0, BUFFER_SIZE);

        sscanf(buffer, " %[^|]|%[^\n]", command, text);

        if (strcmp(command, "UP") == 0)
        {
            to_upper(text);
        }
        else if (strcmp(command, "LOW") == 0)
        {
            to_lower(text);
        }
        else if (strcmp(command, "REV") == 0)
        {
            reverse(text);
        }
        else
        {
            strcpy(text, "Invalid Command");
        }

        send(new_socket, text, strlen(text), 0);
    }

    close(new_socket);
    close(server_fd);

    printf("Server terminated.\n");

    return 0;
}
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>

#define PORT 8081
#define BUFFER_SIZE 1024

int main()
{
    int sock = 0;
    struct sockaddr_in serv_addr;

    char buffer[BUFFER_SIZE];
    char input[BUFFER_SIZE];

    sock = socket(AF_INET, SOCK_STREAM, 0);

    if (sock < 0)
    {
        printf("Socket creation error\n");
        return -1;
    }

    serv_addr.sin_family = AF_INET;
    serv_addr.sin_port = htons(PORT);

    if (inet_pton(AF_INET, "127.0.0.1", &serv_addr.sin_addr) <= 0)
    {
        printf("Invalid address\n");
        return -1;
    }

    if (connect(sock, (struct sockaddr *)&serv_addr,
                sizeof(serv_addr)) < 0)
    {
        printf("Connection Failed\n");
        return -1;
    }

    printf("Connected to server.\n");

    while (1)
    {
        printf("\nEnter command (UP|text, LOW|text, REV|text) or 'bye':\n");

        fgets(input, BUFFER_SIZE, stdin);

        send(sock, input, strlen(input), 0);

        if (strncmp(input, "bye", 3) == 0)
            break;

        memset(buffer, 0, BUFFER_SIZE);

        read(sock, buffer, BUFFER_SIZE);

        printf("Server Response: %s\n", buffer);
    }

    close(sock);

    return 0;
}
