# Crypto-Monitor
Marcela Stade Carvalho - RM552372

# 📱 A aplicação:
**Crypto-Monitor** é um aplicativo Android que exibe a **cotação do Bitcoin em tempo real**. Ao clicar no botão **"Atualizar"**, o valor atual da criptomoeda é mostrado na tela, junto com a data e o horário da última atualização.

| Antes de atualizar | Após atualizar |
|--------------------|----------------|
| ![celular-sem-valor](https://github.com/user-attachments/assets/7f15065e-6056-48ce-94af-c966b764f2be) | ![celular-com-valor](https://github.com/user-attachments/assets/a2dc64c3-21d4-417f-a385-adb489c5f403) |

---

# ⚙️ Funcionamento:
A aplicação foi criada utilizando o ambiente Android Studio, e utiliza da API do Mercado Bitcoin
https://www.mercadobitcoin.com.br/ para buscar a cotação atual da criptomoeda;

A chamada para obter os dados atualizados da API do Mercado Bitcoin ocorre em uma função chamada makeRestCall(), encontrada 
no código principal da aplicação MainActivity.kt. Nele, conseguimos ver o que acontece quando CoroutineScope(Dispatchers.Main).launch 
é chamada, e roda a requisição para a API de forma assíncrona 

Se a resposta tiver sucesso: 
  - extrai o último valor do Bitcoin e converte para Double;
  - 💵 formata o valor para moeda brasileira (R$) e exibe no TextView lbl_value;
  - 📆 converte a data em timestamp Unix para o formato "dd/MM/yyyy HH:mm:ss" e exibe no TextView lbl_date

Se a reposta tiver erro:
  - 4️⃣0️⃣4️⃣ exibe uma mensagem Toast com o erro apropriado: (HTTP 400, 401, 403, 404);
  - ❌ se houver falha na chamada, mostra um Toast com a mensagem da exceção
    
    ```kotlin
      private fun makeRestCall() {
              CoroutineScope(Dispatchers.Main).launch {
                  try {
                      val service = MercadoBitcoinServiceFactory().create()
                      val response = service.getTicker()

              if (response.isSuccessful) {
                  val tickerResponse = response.body()

                  // Atualizando os componentes TextView
                  val lblValue: TextView = findViewById(R.id.lbl_value)
                  val lblDate: TextView = findViewById(R.id.lbl_date)

                  val lastValue = tickerResponse?.ticker?.last?.toDoubleOrNull()
                  if (lastValue != null) {
                      val numberFormat = NumberFormat.getCurrencyInstance(Locale("pt", "BR"))
                      lblValue.text = numberFormat.format(lastValue)
                  }

                  val date = tickerResponse?.ticker?.date?.let { Date(it * 1000L) }
                  val sdf = SimpleDateFormat("dd/MM/yyyy HH:mm:ss", Locale.getDefault())
                  lblDate.text = sdf.format(date)

              } else {
                  // Trate o erro de resposta não bem-sucedida
                  val errorMessage = when (response.code()) {
                      400 -> "Bad Request"
                      401 -> "Unauthorized"
                      403 -> "Forbidden"
                      404 -> "Not Found"
                      else -> "Unknown error"
                  }
                  Toast.makeText(this@MainActivity, errorMessage, Toast.LENGTH_LONG).show()
              }

          } catch (e: Exception) {
              // Trate o erro de falha na chamada
              Toast.makeText(this@MainActivity, "Falha na chamada: ${e.message}", Toast.LENGTH_LONG).show()
          }
      }
    ```
    
    - # 📩 TickerResponse:
      Esse trecho de código Kotlin define duas classes que representam a estrutura da resposta da API do Mercado Bitcoin.

      ```kotlin
      class TickerResponse(
          val ticker: Ticker
      )
      
      class Ticker(
          val high: String,
          val low: String,
          val vol: String,
          val last: String,
          val buy: String,
          val sell: String,
          val date: Long
      )
    
      
     - # 🏭 MercadoBitcoinServiceFactory:
       A classe MercadoBitcoinServiceFactory é utilizada para criar e configurar o Retrofit.
   
       O que é o Retrofit?
       
       O Retrofit é uma biblioteca cliente HTTP para Android e Java/Kotlin, que simplifica a comunicação com APIs REST, permitindo transformar respostas JSON diretamente em objetos Kotlin por meio de interfaces.

       Para isso, utiliza:
    
        - a URL da API (https://www.mercadobitcoin.net/);
        - o conversor JSON GsonConverterFactory
    
        ```kotlin
        class MercadoBitcoinServiceFactory {
    
          fun create(): MercadoBitcoinService {
              val retrofit = Retrofit.Builder()
                  .baseUrl("https://www.mercadobitcoin.net/")
                  .addConverterFactory(GsonConverterFactory.create())
                  .build()
      
              return retrofit.create(MercadoBitcoinService::class.java)
          }
        }
        
    - # 🔁 MercadoBitcoinService:
      A interface Retrofit MercadoBitcoinService é responsável por realizar uma requisição GET para obter a cotação atual do Bitcoin a partir da API do Mercado Bitcoin
      
      Nesse caso, a resposta JSON é convertida automaticamente em um TickerResponse

       ```kotlin
      interface MercadoBitcoinService {
          @GET("api/BTC/ticker/")
          suspend fun getTicker(): Response<TickerResponse>
      }
      
    - # ‼️ Componentes de interface importantes (layout XML):
      Além de todas as classes e interfaces já mencionadas acima, outros arquivos que são de extrema importância para que o funcionamento da aplicação ocorra por completo são:
      - activity_main.xml: responsável por armazenar o código de layout da tela principal;
   
        ```kotlin
        <!-- res/layout/activity_main.xml -->
        <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:orientation="vertical">
        
            <include
                android:id="@+id/component_toolbar"
                layout="@layout/component_toolbar_main"
                android:layout_width="match_parent"
                android:layout_height="75dp"
                android:layout_weight="0" />
        
            <include
                android:id="@+id/component_quote_information"
                layout="@layout/component_quote_information"
                android:layout_width="match_parent"
                android:layout_height="0dp"
                android:layout_weight="1" />
        </LinearLayout>
     
      - component_button_refresh.xml: responsável por criar o botão "Atualizar" e posicioná-lo na tela
      - component_quote_information.xml: responsável por criar o título "Cotação - BITCOIN", campo onde aparecerá o valor de cotação do Bitcoin e campo de data e horário
      - component_toolbar_main.xml: responsável por criar o componente da barra superior (Toolbar)
