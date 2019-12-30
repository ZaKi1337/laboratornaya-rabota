# laboratornaya-rabota
#pragma warning (disable : 4996)
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define LEN 50
#define BLOCK 20

#define FALSE 0
#define TRUE 1

typedef struct PRODUCT
{
	float price;
	char name[LEN];
	char category[LEN];
}PRODUCT;
PRODUCT **list = NULL;
int listlen = 0;

void getDataBaseFromFile(FILE *fd);
void putDataBaseInFile(FILE *fd);
void addProduct(void);
void changeProduct(void);
void removeProduct(void);
void sortList(int mode);
void basketSelection(void);	

int searchProduct(char *str1, char *str2);
void printList(void);

int main(void)
{
	int choise = 0;
	FILE *datfile = NULL;

	system("chcp 1251 > nul");
	printf("Загрузка базы из файла\n");
	if ((datfile = fopen("ProductFile.data", "a+")) == NULL)
	{
		printf("Не удалось открыть файл с базой данных\n");
		system("pause");
		return -1;
	}
	else
	{
		getDataBaseFromFile(datfile);
		fclose(datfile);
		printf("Загрузка базы успешна\n");
		system("pause");
	}
	system("cls");

	while (choise != 7)
	{
		printf( "Меню:\n"
				"1. Добавить товар в базу\n"
				"2. Изменить информацию о товаре\n"
				"3. Удалить товар из базы\n"
				"4. Вывод списка товаров в алфавитном порядке по названию\n"
				"5. Вывод списка товаров в алфавитном порядке по категории\n"
				"6. Подбор минимальной корзины\n"
				"7. Завершение работы\n > ");
		scanf_s("%d", &choise);
		getchar();
		printf("\n");

		switch (choise)
		{
		case 1:
			addProduct();
			break;
		case 2:
			changeProduct();
			break;
		case 3:
			removeProduct();
			break;
		case 4:
			sortList(0);
			printList();
			break;
		case 5:
			sortList(1);
			printList();
			break;
		case 6:
			basketSelection();
			break;
		case 7:
			printf("Сохранение данных в файл и завершение работы\n");
			datfile = fopen("ProductFile.data", "w");
			putDataBaseInFile(datfile);
			fclose(datfile);
			break;
		default:
			printf("Выбран неверный пункт, повторите ввод\n");
			break;
		}
		system("pause");
		system("cls");
	}


	return 0;
}

void getDataBaseFromFile(FILE *fd)
{
	PRODUCT *tmp = malloc(sizeof(PRODUCT));

	while (fread(tmp, sizeof(PRODUCT), 1, fd) != 0)
	{
		if (listlen % BLOCK == 0)
			list = (PRODUCT**)realloc(list, (listlen + BLOCK) * sizeof(PRODUCT*));

		list[listlen++] = tmp;
		tmp = NULL;
		tmp = malloc(sizeof(PRODUCT));
	}
	list = (PRODUCT**)realloc(list, listlen * sizeof(PRODUCT*));
	free(tmp);
}

void putDataBaseInFile(FILE *fd)
{
	int i;
	for (i = 0; i < listlen; i++)
	{
		fwrite(list[i], sizeof(PRODUCT), 1, fd);
		free(list[i]);
	}
	free(list);
}

void addProduct(void)
{
	PRODUCT *tmp = malloc(sizeof(PRODUCT));
	list = (PRODUCT**)realloc(list, (listlen + 1) * sizeof(PRODUCT*));

	printf("Введите название товара\n > ");
	fgets(tmp->name, LEN - 1, stdin);
	printf("Введите категорию товара\n > ");
	fgets(tmp->category, LEN - 1, stdin);
	do {
		printf("Введите стоимость товара, в виде РУБ.КОП\n > ");
		scanf_s("%f", &tmp->price);
		getchar();
	} while (tmp->price < 0.01);
	printf("Запись успешно добавлена\n");
	
	list[listlen++] = tmp;
	tmp = NULL;
}

void changeProduct(void)
{
	int choise = 0, pos;
	char category[LEN], name[LEN];
	PRODUCT *tmp = malloc(sizeof(PRODUCT));

	printf("Введите название товара\n");
	fgets(name, LEN - 1, stdin);
	printf("Введите категорию товара\n");
	fgets(category, LEN - 1, stdin);

	printf("\nИдет поиск товара\n");
	if ((pos = searchProduct(name, category)) == -1)
	{
		printf("\nТавар с заданными параметрами не найден\n");
		return;
	}
	printf("\nТовар успешно найден в базе\n");
	tmp = list[pos];
	list[pos] = NULL;

	while (choise != 4)
	{
		printf("\nИнформация о товаре:\n");
		printf("Наименование товара: ");
		fputs(tmp->name, stdout);
		printf("Стоимость товара: %.2f\n", tmp->price);
		printf("Категория товара: ");
		fputs(tmp->category, stdout);
		
		printf( "\nВыберите нужный пункт\n"
				"1. Изменить цену товара\n"
				"2. Изменить название товара\n"
				"3. Изменить категорию товара\n"
				"4. Завершить изменения\n > ");
		scanf_s("%d", &choise);
		getchar();
		printf("\n");

		switch (choise)
		{
		case 1:
			do {
				printf("Введите новую цену для товара\n > ");
				scanf_s("%f", &tmp->price);
				getchar();
			} while (tmp->price < 0.01);
			system("pause");
			system("cls");
			break;
		case 2:
			printf("Введите новое название для товара\n > ");
			fgets(tmp->name, LEN - 1, stdin);
			system("pause");
			system("cls");
			break;
		case 3:
			printf("Введите новую категорию для товара\n > ");
			fgets(tmp->category, LEN - 1, stdin);
			system("pause");
			system("cls");
			break;
		case 4:
			printf("Информация успешно обновлена\n");
			list[pos] = tmp;
			tmp = NULL;
			break;
		default:
			printf("Выбран неверный пункт, повторите ввод\n");
			system("pause");
			system("cls");
			break;
		}
	}
}

void removeProduct(void)
{
	int pos;
	char name[LEN], category[LEN];

	printf("Введите название товара\n");
	fgets(name, LEN - 1, stdin);
	printf("Введите категорию товара\n");
	fgets(category, LEN - 1, stdin);

	printf("\nИдет поиск товара\n");
	if ((pos = searchProduct(name, category)) == -1)
	{
		printf("\nТавар с заданными параметрами не найден\n");
		return;
	}
	printf("\nИнформация о товаре:\n");
	printf("Наименование товара: ");
	fputs(list[pos]->name, stdout);
	printf("Стоимость товара: %.2f\n", list[pos]->price);
	printf("Категория товара: ");
	fputs(list[pos]->category, stdout);

	list[pos] = list[--listlen];
	list = (PRODUCT**)realloc(list, listlen * sizeof(PRODUCT*));
	printf("\nТовар успешно удален\n");
}

void sortList(int mode)
{
	int i, j, flag = FALSE;
	PRODUCT *tmp = NULL;

	for (i = 0; i < listlen && flag == FALSE; i++)
	{
		flag = TRUE;
		for (j = 0; j < listlen - i - 1; j++)
		{
			if (mode == 0 && strcmp(list[j]->name, list[j + 1]->name) == 1)
			{
				tmp = list[j];
				list[j] = list[j + 1];
				list[j + 1] = tmp;
				tmp = NULL;
				flag = FALSE;
			}
			else if(mode == 1 && strcmp(list[j]->category, list[j + 1]->category)==1)
			{
				tmp = list[j];
				list[j] = list[j + 1];
				list[j + 1] = tmp;
				tmp = NULL;
				flag = FALSE;
			}
		}
	}
}

void basketSelection(void)
{
	char *keystr = NULL, *word = NULL, c;
	int strlen = 0, i, min, pos, flag = FALSE;

	printf("\nВведите категории товаров через пробел, для подбора минимальной корзины\n > ");
	while ((c = getchar()) != '\n')
	{
		if (c == ' ' || c == '\t')
		{
			if (flag == FALSE)
			{
				keystr = (char*)realloc(keystr, (strlen + 2) * sizeof(char));
				keystr[strlen++] = '\n';
				keystr[strlen++] = ' ';
				flag = TRUE;
			}
		}
		else
		{
			keystr = (char*)realloc(keystr, (strlen + 1) * sizeof(char));
			keystr[strlen++] = c;
			flag = FALSE;
		}
	}
	keystr = (char*)realloc(keystr, (strlen + 2) * sizeof(char));
	keystr[strlen++] = '\n';
	keystr[strlen] = '\0';

	word = strtok(keystr, " ");
	while (word != NULL)
	{
		min = INT_MAX;
		pos = -1;
		for (i = 0; i < listlen; i++)
		{
			if (strcmp(word, list[i]->category) == 0)
			{
				if ((int)(list[i]->price * 100) < min)
				{
					min = (int)(list[i]->price * 100);
					pos = i;
				}
			}
		}
		printf("\nТовар из категории %s", word);
		if (pos == -1)
		{
			printf("Товар не найден\n");
		}
		else
		{
			printf("Наименование товара: ");
			fputs(list[pos]->name, stdout);
			printf("Стоимость товара: %.2f\n", list[pos]->price);
			printf("Категория товара: ");
			fputs(list[pos]->category, stdout);
		}

		word = strtok(NULL, " \t");
	}
}


int searchProduct(char *name, char *category)
{
	int i;
	for (i = 0; i < listlen; i++)
		if (strcmp(list[i]->name, name) == 0 && strcmp(list[i]->category, category) == 0)
			return i;
	return -1;
}

void printList(void)
{
	int i;
	printf("Список товаров в базе:\n");
	for (i = 0; i < listlen; i++)
	{
		printf("\nНаименование товара: ");
		fputs(list[i]->name, stdout);
		printf("Стоимость товара: %.2f\n", list[i]->price);
		printf("Категория товара: ");
		fputs(list[i]->category, stdout);
	}
} 
