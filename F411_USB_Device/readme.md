# Activation du port USB en mode Device

## Références

https://controllerstech.com/send-and-receive-data-to-pc-without-uart-stm32-usb-com/



## Code associé 

Pour transmettre/recevoir implémenté le code suivant

**usbd_cdc_if.h**  

```C
/* USER CODE BEGIN EXPORTED_TYPES */
 typedef void (*USB_DataReceived_Callback)(uint8_t *data, uint32_t length);
/* USER CODE END EXPORTED_TYPES */

/* USER CODE BEGIN EXPORTED_FUNCTIONS */
void USB_Register_DataReceived_Callback(USB_DataReceived_Callback callback);
/* USER CODE END EXPORTED_FUNCTIONS */
```

**usbd_cdc_if.c**

```
Ajouter
/* USER CODE BEGIN PRIVATE_FUNCTIONS_IMPLEMENTATION */
void USB_Register_DataReceived_Callback(USB_DataReceived_Callback callback) {
	data_received_callback = callback;
}
/* USER CODE END PRIVATE_FUNCTIONS_IMPLEMENTATION */

Modifier
static int8_t CDC_Receive_FS(uint8_t *Buf, uint32_t *Len) {
	/* USER CODE BEGIN 6 */
	USBD_CDC_SetRxBuffer(&hUsbDeviceFS, &Buf[0]);
	USBD_CDC_ReceivePacket(&hUsbDeviceFS);

	// Appelle le callback utilisateur si enregistré
	if (data_received_callback != NULL) {
		data_received_callback(Buf, *Len);
	}

	return (USBD_OK);
	/* USER CODE END 6 */
}
```

**main.c**

```C
/* USER CODE BEGIN Includes */
#include "usbd_cdc_if.h"
/* USER CODE END Includes */

/* Private user code ---------------------------------------------------------*/
/* USER CODE BEGIN 0 */
/**********************************************
 * Fonction de traitement des données reçues
 **********************************************/
void My_USB_DataReceived_Handler(uint8_t *data, uint32_t length) {
	// Exemple : Afficher les données reçues ou les renvoyer
	if(APP_RX_DATA_SIZE > length + 2){
		strcat((char*)data, "\n\r");
	}
	CDC_Transmit_FS(data, length);ansmit_FS(data, length);
}
/* USER CODE END 0 */

int main(void) {
    ...
    
    	/* USER CODE BEGIN 2 */
	// Enregistrer le callback pour traiter les données reçues
	USB_Register_DataReceived_Callback(My_USB_DataReceived_Handler);
	/* USER CODE END 2 */

	/* Infinite loop */
	/* USER CODE BEGIN WHILE */
	char testDataToSend[100];
	int counter = 0;

	while (1) {
		/* USER CODE END WHILE */

		/* USER CODE BEGIN 3 */
		HAL_Delay(1000);
		sprintf(testDataToSend, "Hello World %d\n\r", counter++);
		CDC_Transmit_FS((uint8_t*) testDataToSend, strlen(testDataToSend));

	}
	/* USER CODE END 3 */
}
```

