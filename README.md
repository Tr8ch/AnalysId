# Отчет работы и результата сервиса idocs.kz

> 
> Результатом анализа является ответ на вопрос:
> - Как интегрировать подписи системы, так чтобы они могли проверяться на check.doodocs.kz?
> 
#### Какой тип подписи? - XML, CMS, etc.
> - `.xml` файл, каждого подписанта
#### Как выглядит подписанный документ, который подтверждает факт успешного подписания?
> - Это `.zip` архив, который содержит в себе папку (с наванием подписываемого документа и номером документа, котрый вы указываете на сайте). Непосредственно в этой папке находятся 3 файла:
> 	- Оригинал документа
>	- Паспорт документа (`.pdf` документ, содержащий в себе информацию о подписантах и изменениях в отправке документа)
>	-  Печатная форма вашего документа
> - Еше N (кол-во подписантов) директорий (папок) в каждой из которых лежит подпись подписанта в формате `.xml`
![Содержание](/img/Soderzhanie.png)

#### Что конкретно подписывается? - Хеш, бинарные данные, UUID документ и т.д.
> - Хэш вашего документа 
#### Как проверяется на сервисе?
> - Непосредственно сервиса проверки подписанного документа на легитимность нет. Но на странице с документом может быть флаг "Подписан/Ожидает подписания/Отправлен/Черновик" что и являеся их проверкой.
![Подписан](/img/Podpisan.png)
![Ожидает подписания](/img/Ozhidaet.png)
![Отправлен](/img/Otpravlen.png)
![Черновик](/img/Chernovik.png)

#### Как можно проверить подпись вне этого сервиса?
> - Проверить легитимность подписи можно на сайте https://ezsigner.kz/#!/checkCMS, отправив `.xml` подпись каждого подписанта отдельно. 

## HTTP-запросы

> \*Данный текст не является частью анализа и нужно удалить при сдаче.
> 
> В этом разделе нужно описать последовательность HTTP запросов, которые приводят к подписанию документа.
> Каждый запрос должен иметь пример HTTP-запроса командой `curl` и HTTP-ответ.
> Каждый запрос должен быть описан, где объясняется что происходит и зачем нужен этот запрос.
> 
> Ниже представлены примеры.


### №1. Загрузка документа

- Данный ниже `curl` запрос может отправить документ и создать его в разделе "Внешние/Черновики", в ответ вы получите json response о успешном создании документа.
- Так же вы можете заметить в комментариях флаг `{omitempty}` если убрать поля под ним и отправить запрос без них, то в ответ так же придет json response о успешном создании документа, но в разделе "Внешние/Черновики" документ не появится:)
- При отправке запроса не забудьте заменить "`...`" на значения подходящего типа.
- Указать полный путь вместо `<filepath>`.
- А также если установить значения в поле `Category=0`, то в ответ так же придет json response о успешном создании документа, но в разделе "Внешние/Черновики" документ не появится:) 
- Если установить `CreationType` в значении `2`, то документ можно будет просмотреть и при его скачивании никаких проблем не возникает, а вот в остальных значенияx `1/3/4` документ нельзя просмотреть и при скачивании выдает ошибку или скачивает пустой архив.
- `DocumentGroupId` есть 10 значений для каждого вида докумена:

 Юридические  
 - a3fa4c08-34fa-428d-afc5-7073c9fcab69
   
 Бухгалтерские
 - 4484d2e5-ec9c-4715-8ac9-46dc90e32594

 Складские
 - ab79b353-820b-4afd-87dd-4e8e10f633bf

 Кадровые
 - eb1d1a24-5d62-4db5-9586-2591d2d195b7

 Служебные
 - 50e5f2fb-b93a-476d-9ac9-84c5fa29d584

 Финансовые 
 - d5bbceda-3276-483a-8688-da77d8427b28

 Учредительные
 - ffd87147-84f4-41a9-ad39-fba1cf200e9c

 Организационно-правовые
 - 8cf4366c-ece9-41e5-862d-cfbcd4ae60d8

 Закупки
 - c2d3f988-c359-42f7-bc7f-b2b459b1bc27
 
 Прочие
 - 0cea3b9b-ac5e-407e-a79e-6292df9d4981


#### `POST` Запрос:
```sh
curl --location 'https://api.idocs.kz/api/v2/reference/fact-documents/iDocsApi.create-with-file' \
--header 'Authorization: Bearer Token ...' \
# Number string {omitempty}
--form 'Number="..."' \
# Name string
--form 'Name="..."' \
# {omitempty}
--form 'Date="YYYY-MM-DDTHH:MM:SS.sssZ"' \
# CreationType int32 от 0 до 4 {omitempty}
--form 'CreationType="..."' \ 
# {omitempty}
--form 'File=@"<filepath>"' \
# Category int32 есть два значения (0 и 1)
--form 'Category="..."' \
# DocumentGroupId uuid
--form 'DocumentGroupId="..."'
```

Ответ:

```json
{
    "@odata.context": "https://api.idocs.kz/api/v2/reference/$metadata#iDocsApi.ApiSuccessResponseModel_1OfGuid",
    "Header": "Успешно",
    "Description": "Документ с номером ... успешно создан",
    "Payload": "e56a18df-1c27-4df3-314b-08dbe238765a"
}
```

### №2. Отправка подписи

Для того чтобы отправить подпись нужно загрузить документ (с помощью `curl` запроса выше). Взять с ответа значение(`UUID`) с ключа `"Payload"`

Потом отправить `curl` запрос подписания документа, он выдаст нам `.xml` файл такого вида:

```http
<root><data>XXX<data><timestamp>YYY</timestamp></root>

```
 curl --location --request GET 'https://api.idocs.kz/api/v2/sign/8e5712e0-a303-4ee4-7df8-08dbe3161733' \
   --header 'Accept: application/json, text/plain, */*' \
   --header 'Authorization: Bearer eyJhbGciOiJSU0EtT0FFUCIsImVuYyI6IkEyNTZDQkMtSFM1MTIiLCJraWQiOiI2MjRCQjk2OTMwQzNBMDBFNTE2NEU5Qjg2Q0M5MkQwOTMzQ0ZFMUMyIiwidHlwIjoiYXQrand0In0.aMwvNxQ6JvaB2tbqReuiNFi-c7wbLTQmHu63V-eRSFB7RseQ1v8sJikbd2jylzhjEMMwiEKNtVA4JeVI5i4mpYS9COxVG-m80DoiZ_Yyi94B9K9P62mEwQQ9EnE9MUxXL-BMhsqG08Mp5Ivxn0bqeOg5MhQic-GaKCHxuUWubJuI2SbChTyjWeWuHkcGlLoedtUYbj-6Uoet0cmsETBa06Ql3kSKGB8iDpH90KgZRXZsEcQWM7TqSZ_86TJMq8--__vtXgsj0WINYgiioyzLmWcF6kPj5ELKvcBZulTZBgEM24d5ZiiHNNwLnzgclaC2WwkU4rWH1SKfsXvuSH9f4Q.Wcikxolj7KCMNugNsuqeGg.lYOFBZPmUSIAGIJ8Ts_yD1-znyGaD9tQV77YmaAY84oum5snvE2wrbDJMhs9jQrNT5RHP1vZrIEk1vEa_D5Zeeobd1YeWx2pP4LP1jgwFBd5PHjfxotYNxRdYnqKhN6jPrP_w4CRn8Op7Tbk_tWM8KsNFBCoY-IqVtrjQ-x-Gz_hjNex2eKDZb-TutC7xJ-EohRXD4bnX5ganiOPxfGk-DPXA9tUyFToZjnD3EHwdaaQe0AqFRsc88o3ON98BxCh9J7ttPVgknyIeumfqYd1MagBiwvQpCwZGWj1kAfcvPYtxnd3C92aR0Nw5qPdBAvFOpZ2ynNPPhrCoP28zIOuWPNaX5O5bUfHqRzLD9ueoyyQfInO8vojXPjL9nSwQADRY85B_3rCQnYr9YuCGrQUz5YPMPchLXPYa5Y9aZP-Z-Pe4LHimGDemiPlAqthVjAiBIvM2U1d_FxM7FuDk_3rmpq61vUGQ8flq4YNB8UrEA-iy0nvjnJT0GnZV2wk0kCmF7eddyYoxqdZxQVsyHxPM7fDpxG1Mu0eU5bbZblZj2liORPvQTDTDE3KdhsLMlW-Zk8zJ5kWQBtSxK3puYgtRwIg347UIo8-mSmrl5tYyoyXdYftDJ69_x3y31LnOK5YLCSuVnmbQ1gMN9WqJk-VmmcakLYPVVsJX4anYCSkEcjCJRPnCEjmN8JTBDLD3IvhcKnRIJCpUMcSSIKl3cfUBVLSBrHDmo2OpNrd7GJjfG2coHizi9eApLRAXojMZ_lAelJP0qlAw7icNi4EplCwy6YcQ3C-VqF9-9Gq95iNUyd5LeOCs374X-A1ImHDO6q-sDGsxi-aX2M4fDyPnODAiiNBiqsDFAI6MV3D3v5epzq842q3qqCDZ6vf10PXtI5L8dvvbbuh4bPN8G_B55BW9rKNbBp2gCkyJhFBX3gJO1ZtfiolZdHtwhj6eK2QSIuv7F4pYAAs9_WfGtDzeQUmfrxZP2ecUgWV8moPsbJ_iPgQE_0HdKTY2wdQnlhu_FvBLe6az0vkkS09G9iG9j1KVgv8MzzAb60vxRcc86zT9FYaXzWgB6gt6rZSFdXjpfP4S0J4APNB2b8g3mpRKkQfxPqW0Bz6nXQ3LH_ot3Kst7SLb9XFjIHOmF8CDIeGIUF7Wooa01Dt-qYjL6hE3oABeso0_AR7s722ooLAtdSnzMyD1dqsLSnIsPDAIXyWRT4AfWNan0JXn_jMpBLgfTTQUiMYmsRaeGJ4ZQ6oi8urQ6eu8PoF_mhscoaA7YuJtR-JC9kNRAgGKyFycj95noxknw7XNH1bo1W45PBV2i2_KfU3U1NTPZnjZ6Y-bpjrgkCKjMxnB94pZ8UpjPDn70VBS5KPuqza7FB7qRhNxDVfjb-pcjC0QYHvnb8Y3_sANdVP-L8jiavqnvsSVbap7pHx_v2Hfs0MS8RKFuQ_7_9ZgTi37mGQ_5KTf--eaDXRIz0tNLIF3ILJiEGW9iK4mgqBstol_ktY5UlUjFHzVPpewqyWqCTI20HDhtmvboTALqUM4Wm0TjFDK77bOPrc6KXHJA2BEgFv2nx-VImDDKpPY6ly04X0-96fnZlshgw-aez2vRtoTLEtbhIsxHiIQ7NgEvEMTGFrsguJhhtkxXrWezUadC7UOmnMYzT4rnyx5to2ktg3AHJctZn-n2WaXlxfl5WCaNqzL9hd9wzzTXnA9GlFmAHJ6-uU8qmzrrNg0jU8qcLc0mkcZuxrdettWQXkhMzaf1O1ETVOqr29OYyOuvCG4OQdQgpQJOH4Ou_FeqCqUTP9IgXRW3px0Yob8kjtXy3Y2U1Rhk_hVMH9ZNmvkX3_WBFt2yCbIzaifNu9OcqTzA7hkajFqrsJQskbWk83Xca4RCiEpXfxr-R0iCEHbPW86bnZ_LeVMvrIy09wzinQ_n7ibzxZJapUvpKDPj22BJ9ftnnMOMOsMuAiPhUo3XPnXozfLG3RAaULP6_93OUQw9nAP6ciV-_dYLJoqAR6LYB3uKk1GS0Qd9j91t8aZPxe5flLwoW_FloimiMpiPmAtxlKpD_K33QIZXpnHFT8paobYGitdkJVRsOaKp4a1Dj1D0zFbpvimKE3rqhNnia8zqVvchoFIYn1CeYzIJui3_DMGJWA9gYAKJEK42ou17rrFJcKMWcZftC_jrWKwUDIzUKj2p9UE3uhL0XcvOsH6AZniiPhvNO9Buq9VRB8ux-tAB8ErrNPCnJuldCzQ1edqqTAOoNSbRH8TbKyJDULrUG_XjinR9ZwPmBkzaOThGnrs1LC8R3ugO_9SonHJpjtmJI8Bcae7XCZLenUpVuxojgAZS_6ESxm08BxB9JOKul1q8_CtbgFd_Bq_2wvQY3obBJmLqjEqs9NdniMgD6HanyPRgLO1GSJZ--WGoevQh_Fp-uw6-UP8_AtvynJPwr_dIXojkE_X36uQiU19roouLKLUVh_55EalMEvhnoeVnUO6oH0Qpm64qvYGo-Mzu5c_6aWmKBHOVHJ0WJ60-Isld5M8VBNLxUQe31AuxQB1QZJ7bHq24yyUgT68RdF1bCkDYbEH-7hLLtp3XEVBVHN1TLCzGzFrvI40cyWsurdmVYpEshdfz-cAqf7KqQyPFeG0OdcQfCRYXCpryLgNH3MJLKVaQ2Lt73ns5gCPTh_Li6IgcekrdbEJdg81ydBhUZNSOndN87cj_x0zsoincxZHRkXb1C76S9oDTazjcl5OnTmb4aW70hSxTv-PcYwvCqjuyPuVYUvlxyqzPRHXUOVQl4G29eBAQOF-mmB95W4_vK1CRwjWhThuXTqki8Ol_IVZie25YoEKsRgRRzy0rfhhNSJl7AON-eYnh0U6jtIlDtr0GSI3MAgNosJsEh66R8b2f3hXJg4IAw-4v9kld8jSN_el59kE2A_WYmxpMy04acXK1Pqnyuor_RvXGB36TCVNDRWVFx80y7LoJLqNXpceqn1EDnN7CmE9K4KsMoNkqrCRT3VYmt0br0q5JvsHLRCzzso9XkqFFaITjoh_efN9wd7kj5uHqmHai3Tblo3n0VDIybtBxQtM1IYb6bhuXUeug4aSLuGmkNodrYeA5SRkyL2eeKEFo0mha2F_aov-q_hh6iX0S4EGn4sEpjPM4ZWvuvT7wOxSFXTg9ZScLOJ4g3-zti27u4hqIvezsZfOoGqWbnpe4hLUxC5E5DM7ignAkDC0WQXyGWT2URU-_7yt9AG4bKooWqBmjsNAAmRo3H-MRpfjHonLCIhBCDcY6CQdO5sYJiVURx-Oe_eIzG6A1ETYYOuTw6vdNLXzAvRR_xFbo8UNqfZOEtzvEj2T2q4p0sozQJ3yKwhQMW-PSfCoz0jLWTBF4eOirI0rjW1zAB2TMm33eysC3op9ZpKdb2hkkTYbgPTxHlKi0qyaxPRN57WAT2AsI-RvVRLzqHkMkescuiF7kkwxRRvKIpqkNv-AxvlBgm0MlivvTxJ3cSISAds0LUHqVyHlw_FsH4as04nzUl4z4luUJBKxZeP9cOXg8eLyJ6rYMP-Qp86EoTLuqUKGqnAolEjVh0FGjJsxG8UTcUWgyqL0c6DOrz8w1nwoN5EfWDazDB8jfYMQGRRBe66lpVT7k5oTaj-NcCSHTycbB13C4PJd96AReIfrLVVnJHdmaGJaLPaQ_2yMBWIRJ2r6fqSxk4d5PbEPxrBNE15DE__quGhKrzHPz2J7jOIocNfGSkYtIwD2SrONs3JXeTh0ORkZSxp8xlr4RBCaWMfgit0Syg5wHPLu4ev5F-JFRPneWVIOXa1fxrRm1f4_4LTIGSSCSAM3w5nk5auKhiTLw3ydN06gyvREWpmk_CROWH4p6AsVxWapUmnT6tqLBQh5-_ltS6qWuTwbHGDCVMtkRs68_BEj8nKn5qDok3FLz-FKG6NNy7muUFvM7Wfrxf1Yo6U9ubp4_0wvqezXCDh-pkfnQJFrfaQo-p0fkw9aOrJ2R4Uc3e2cEbbyQRaiSuHrVbkVaGBOX5OB2cMtGveH1XWpI-jtNeCOzrszDmTbejH2vpyK_zj0k2qRYEk6FMsjh2xrpCs1dGnsavLqYwfRL5iPCtl1J9izBdl1o2YhhsZWoos5fjEctEQdTixrD8fukOsLEHOjeoHObBVV4ohMVaKbQ9Ik_l-8m3PqVe8RZJ08cqfAyNoFVNUx93S_0aXgGAS8CdcCkIbUSqZaB5_M47NsIN7gBG17YcIt8UgVqQ_w3tJmjMfRUDv7VvaGA8-w9FPZa-elEEp4a-WP90h2moSj1PYmrRNRsEedJcyWOioLzYaPQtFIPlm40PrjUkfhY066Q8ohd5RiShgBTAIYdgp3Dsx3iVymZEhVYFBy3VdI39dlRBbM59a_a-X8jk0rlJKuYwoqTHxR1QTFVWtfIpkCMb79Ejr5O-X0yhcMJ67FpILpN9EDdOBy7SJh4slKrlBtxhCtr7wZSgnCL29jq_3ws-V7aT-Wa4uAB6e.GhIAMNjw6k0J1C07we-kaq4_MsXTlAK-9ZLxPErjLcU' \
   --header 'Content-Type: application/json' \
 
Ответ:
  ```json
  {
    "FactDocumentId": "8e5712e0-a303-4ee4-7df8-08dbe3161733",
    "ContentList": [
        {
            "FileId": "aa505328-f6b3-481b-f8ad-08dbe316173b",
            "Content": "<root><data>1F5453A137690E2DA81AA5C552AE9D2D7E175E53F131A89DB338F0E7184B862721DB25FC77E18CF373ACD529AE397F32A37634A21F6DBD0A2E9D98F39E08221C</data><timestamp>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</timestamp></root>",
            "Ticket": "content_ticket_12e52665-229b-4fb1-9245-442ff321d27d"
        }
    ],
    "StepId": null,
    "StageId": null,
    "PositionName": null
}
```





1F5453A137690E2DA81AA5C552AE9D2D7E175E53F131A89DB338F0E7184B862721DB25FC77E18CF373ACD529AE397F32A37634A21F6DBD0A2E9D98F39E08221C
   
Данный запрос отправляет подпись и привязывает его к документу. В ответ не передается никакой информации - только статус 200.

Запрос:
```sh
curl -X POST \
	-H "Content-Type: multipart/form-data" \
	-H "Authorization: Bearer token..." \
	-d "..." \ # Передаваемые данные
	https://api.system.kz/api/v1/prepare/documents/3fec9ab2-3b88-4bf7-ada3-2f427643ba1c/sign
```

```sh
   curl --location --request GET 'https://api.idocs.kz/api/v2/sign/76392aea-8929-4329-7df3-08dbe3161733' \
   --header 'Accept: application/json, text/plain, */*' \
   --header 'Authorization: Bearer eyJhbGciOiJSU0EtT0FFUCIsImVuYyI6IkEyNTZDQkMtSFM1MTIiLCJraWQiOiI2MjRCQjk2OTMwQzNBMDBFNTE2NEU5Qjg2Q0M5MkQwOTMzQ0ZFMUMyIiwidHlwIjoiYXQrand0In0.aMwvNxQ6JvaB2tbqReuiNFi-c7wbLTQmHu63V-eRSFB7RseQ1v8sJikbd2jylzhjEMMwiEKNtVA4JeVI5i4mpYS9COxVG-m80DoiZ_Yyi94B9K9P62mEwQQ9EnE9MUxXL-BMhsqG08Mp5Ivxn0bqeOg5MhQic-GaKCHxuUWubJuI2SbChTyjWeWuHkcGlLoedtUYbj-6Uoet0cmsETBa06Ql3kSKGB8iDpH90KgZRXZsEcQWM7TqSZ_86TJMq8--__vtXgsj0WINYgiioyzLmWcF6kPj5ELKvcBZulTZBgEM24d5ZiiHNNwLnzgclaC2WwkU4rWH1SKfsXvuSH9f4Q.Wcikxolj7KCMNugNsuqeGg.lYOFBZPmUSIAGIJ8Ts_yD1-znyGaD9tQV77YmaAY84oum5snvE2wrbDJMhs9jQrNT5RHP1vZrIEk1vEa_D5Zeeobd1YeWx2pP4LP1jgwFBd5PHjfxotYNxRdYnqKhN6jPrP_w4CRn8Op7Tbk_tWM8KsNFBCoY-IqVtrjQ-x-Gz_hjNex2eKDZb-TutC7xJ-EohRXD4bnX5ganiOPxfGk-DPXA9tUyFToZjnD3EHwdaaQe0AqFRsc88o3ON98BxCh9J7ttPVgknyIeumfqYd1MagBiwvQpCwZGWj1kAfcvPYtxnd3C92aR0Nw5qPdBAvFOpZ2ynNPPhrCoP28zIOuWPNaX5O5bUfHqRzLD9ueoyyQfInO8vojXPjL9nSwQADRY85B_3rCQnYr9YuCGrQUz5YPMPchLXPYa5Y9aZP-Z-Pe4LHimGDemiPlAqthVjAiBIvM2U1d_FxM7FuDk_3rmpq61vUGQ8flq4YNB8UrEA-iy0nvjnJT0GnZV2wk0kCmF7eddyYoxqdZxQVsyHxPM7fDpxG1Mu0eU5bbZblZj2liORPvQTDTDE3KdhsLMlW-Zk8zJ5kWQBtSxK3puYgtRwIg347UIo8-mSmrl5tYyoyXdYftDJ69_x3y31LnOK5YLCSuVnmbQ1gMN9WqJk-VmmcakLYPVVsJX4anYCSkEcjCJRPnCEjmN8JTBDLD3IvhcKnRIJCpUMcSSIKl3cfUBVLSBrHDmo2OpNrd7GJjfG2coHizi9eApLRAXojMZ_lAelJP0qlAw7icNi4EplCwy6YcQ3C-VqF9-9Gq95iNUyd5LeOCs374X-A1ImHDO6q-sDGsxi-aX2M4fDyPnODAiiNBiqsDFAI6MV3D3v5epzq842q3qqCDZ6vf10PXtI5L8dvvbbuh4bPN8G_B55BW9rKNbBp2gCkyJhFBX3gJO1ZtfiolZdHtwhj6eK2QSIuv7F4pYAAs9_WfGtDzeQUmfrxZP2ecUgWV8moPsbJ_iPgQE_0HdKTY2wdQnlhu_FvBLe6az0vkkS09G9iG9j1KVgv8MzzAb60vxRcc86zT9FYaXzWgB6gt6rZSFdXjpfP4S0J4APNB2b8g3mpRKkQfxPqW0Bz6nXQ3LH_ot3Kst7SLb9XFjIHOmF8CDIeGIUF7Wooa01Dt-qYjL6hE3oABeso0_AR7s722ooLAtdSnzMyD1dqsLSnIsPDAIXyWRT4AfWNan0JXn_jMpBLgfTTQUiMYmsRaeGJ4ZQ6oi8urQ6eu8PoF_mhscoaA7YuJtR-JC9kNRAgGKyFycj95noxknw7XNH1bo1W45PBV2i2_KfU3U1NTPZnjZ6Y-bpjrgkCKjMxnB94pZ8UpjPDn70VBS5KPuqza7FB7qRhNxDVfjb-pcjC0QYHvnb8Y3_sANdVP-L8jiavqnvsSVbap7pHx_v2Hfs0MS8RKFuQ_7_9ZgTi37mGQ_5KTf--eaDXRIz0tNLIF3ILJiEGW9iK4mgqBstol_ktY5UlUjFHzVPpewqyWqCTI20HDhtmvboTALqUM4Wm0TjFDK77bOPrc6KXHJA2BEgFv2nx-VImDDKpPY6ly04X0-96fnZlshgw-aez2vRtoTLEtbhIsxHiIQ7NgEvEMTGFrsguJhhtkxXrWezUadC7UOmnMYzT4rnyx5to2ktg3AHJctZn-n2WaXlxfl5WCaNqzL9hd9wzzTXnA9GlFmAHJ6-uU8qmzrrNg0jU8qcLc0mkcZuxrdettWQXkhMzaf1O1ETVOqr29OYyOuvCG4OQdQgpQJOH4Ou_FeqCqUTP9IgXRW3px0Yob8kjtXy3Y2U1Rhk_hVMH9ZNmvkX3_WBFt2yCbIzaifNu9OcqTzA7hkajFqrsJQskbWk83Xca4RCiEpXfxr-R0iCEHbPW86bnZ_LeVMvrIy09wzinQ_n7ibzxZJapUvpKDPj22BJ9ftnnMOMOsMuAiPhUo3XPnXozfLG3RAaULP6_93OUQw9nAP6ciV-_dYLJoqAR6LYB3uKk1GS0Qd9j91t8aZPxe5flLwoW_FloimiMpiPmAtxlKpD_K33QIZXpnHFT8paobYGitdkJVRsOaKp4a1Dj1D0zFbpvimKE3rqhNnia8zqVvchoFIYn1CeYzIJui3_DMGJWA9gYAKJEK42ou17rrFJcKMWcZftC_jrWKwUDIzUKj2p9UE3uhL0XcvOsH6AZniiPhvNO9Buq9VRB8ux-tAB8ErrNPCnJuldCzQ1edqqTAOoNSbRH8TbKyJDULrUG_XjinR9ZwPmBkzaOThGnrs1LC8R3ugO_9SonHJpjtmJI8Bcae7XCZLenUpVuxojgAZS_6ESxm08BxB9JOKul1q8_CtbgFd_Bq_2wvQY3obBJmLqjEqs9NdniMgD6HanyPRgLO1GSJZ--WGoevQh_Fp-uw6-UP8_AtvynJPwr_dIXojkE_X36uQiU19roouLKLUVh_55EalMEvhnoeVnUO6oH0Qpm64qvYGo-Mzu5c_6aWmKBHOVHJ0WJ60-Isld5M8VBNLxUQe31AuxQB1QZJ7bHq24yyUgT68RdF1bCkDYbEH-7hLLtp3XEVBVHN1TLCzGzFrvI40cyWsurdmVYpEshdfz-cAqf7KqQyPFeG0OdcQfCRYXCpryLgNH3MJLKVaQ2Lt73ns5gCPTh_Li6IgcekrdbEJdg81ydBhUZNSOndN87cj_x0zsoincxZHRkXb1C76S9oDTazjcl5OnTmb4aW70hSxTv-PcYwvCqjuyPuVYUvlxyqzPRHXUOVQl4G29eBAQOF-mmB95W4_vK1CRwjWhThuXTqki8Ol_IVZie25YoEKsRgRRzy0rfhhNSJl7AON-eYnh0U6jtIlDtr0GSI3MAgNosJsEh66R8b2f3hXJg4IAw-4v9kld8jSN_el59kE2A_WYmxpMy04acXK1Pqnyuor_RvXGB36TCVNDRWVFx80y7LoJLqNXpceqn1EDnN7CmE9K4KsMoNkqrCRT3VYmt0br0q5JvsHLRCzzso9XkqFFaITjoh_efN9wd7kj5uHqmHai3Tblo3n0VDIybtBxQtM1IYb6bhuXUeug4aSLuGmkNodrYeA5SRkyL2eeKEFo0mha2F_aov-q_hh6iX0S4EGn4sEpjPM4ZWvuvT7wOxSFXTg9ZScLOJ4g3-zti27u4hqIvezsZfOoGqWbnpe4hLUxC5E5DM7ignAkDC0WQXyGWT2URU-_7yt9AG4bKooWqBmjsNAAmRo3H-MRpfjHonLCIhBCDcY6CQdO5sYJiVURx-Oe_eIzG6A1ETYYOuTw6vdNLXzAvRR_xFbo8UNqfZOEtzvEj2T2q4p0sozQJ3yKwhQMW-PSfCoz0jLWTBF4eOirI0rjW1zAB2TMm33eysC3op9ZpKdb2hkkTYbgPTxHlKi0qyaxPRN57WAT2AsI-RvVRLzqHkMkescuiF7kkwxRRvKIpqkNv-AxvlBgm0MlivvTxJ3cSISAds0LUHqVyHlw_FsH4as04nzUl4z4luUJBKxZeP9cOXg8eLyJ6rYMP-Qp86EoTLuqUKGqnAolEjVh0FGjJsxG8UTcUWgyqL0c6DOrz8w1nwoN5EfWDazDB8jfYMQGRRBe66lpVT7k5oTaj-NcCSHTycbB13C4PJd96AReIfrLVVnJHdmaGJaLPaQ_2yMBWIRJ2r6fqSxk4d5PbEPxrBNE15DE__quGhKrzHPz2J7jOIocNfGSkYtIwD2SrONs3JXeTh0ORkZSxp8xlr4RBCaWMfgit0Syg5wHPLu4ev5F-JFRPneWVIOXa1fxrRm1f4_4LTIGSSCSAM3w5nk5auKhiTLw3ydN06gyvREWpmk_CROWH4p6AsVxWapUmnT6tqLBQh5-_ltS6qWuTwbHGDCVMtkRs68_BEj8nKn5qDok3FLz-FKG6NNy7muUFvM7Wfrxf1Yo6U9ubp4_0wvqezXCDh-pkfnQJFrfaQo-p0fkw9aOrJ2R4Uc3e2cEbbyQRaiSuHrVbkVaGBOX5OB2cMtGveH1XWpI-jtNeCOzrszDmTbejH2vpyK_zj0k2qRYEk6FMsjh2xrpCs1dGnsavLqYwfRL5iPCtl1J9izBdl1o2YhhsZWoos5fjEctEQdTixrD8fukOsLEHOjeoHObBVV4ohMVaKbQ9Ik_l-8m3PqVe8RZJ08cqfAyNoFVNUx93S_0aXgGAS8CdcCkIbUSqZaB5_M47NsIN7gBG17YcIt8UgVqQ_w3tJmjMfRUDv7VvaGA8-w9FPZa-elEEp4a-WP90h2moSj1PYmrRNRsEedJcyWOioLzYaPQtFIPlm40PrjUkfhY066Q8ohd5RiShgBTAIYdgp3Dsx3iVymZEhVYFBy3VdI39dlRBbM59a_a-X8jk0rlJKuYwoqTHxR1QTFVWtfIpkCMb79Ejr5O-X0yhcMJ67FpILpN9EDdOBy7SJh4slKrlBtxhCtr7wZSgnCL29jq_3ws-V7aT-Wa4uAB6e.GhIAMNjw6k0J1C07we-kaq4_MsXTlAK-9ZLxPErjLcU' \
   --header 'Content-Type: application/json' \
  
```

Ответ:

```http
200 OK
```

## Описание подписанного документа

1) ***Оригинал документа***
   - Это изначальный документ, который вы как клиент загрузили на сайт и редактировали или же созданный вами документ с помощью редактора/шаблона на сайте.
2) ***Паспорт докмуента***
   - Это `.pdf` файл в котором хранится информация о документе и подписантах. 
   - Сначала идет дата скачивания подписанного файла. 
   - Далее таблица, в первой секции которой содержится небольшая информация о документе: категория, название, ссылка на документ, статус подписи. 
   - В следующей секции описана информация о "Отправителе" документа: наименование компании/ФИО физ. лица и БИН/ИИН (соответсвенно). Так же описана информация о "Получателях". На этом таблица заканчивается. 
    ![Таблица](/img/Table1.png)
   - "Отправитель" и "Получатели" являются подписантами документа. 
   - Далее ниже идут уже детали о подписантах: их ФИО, наименования компаний(если документ подписывали от юр. лица), должности, виды подписей, издатели, серйнные номера сертификатов, дата и время когда подписант подписал документ, действительность сертификата на момент подписания и часть хэша подписи.
     	![Детали](/img/Detali.png)
   - Далее идет история докумета внутрии компании с ФИО сотрудника, датой и временем определенного действия. 
    ![История в компании](/img/History1.png)
   - И в конце описывается история документа по контрагентам также с датой и временем
    ![История по контрагентам](/img/History2.png)
3) ***Печатная форма***
   - Это `.pdf` файл который содержит в себе ваш оригинальный документ, но с добавлением небольшой информации от *idocs*, ссылкой и qr-кодом, содержащий ссылку на просмотр вашего документа в сервисе *idocs*.
   ![Инфо Плашка](/img/InfoPlashka.png)
   - Так же если вы не использовали шаблоны представленные в сервисе, а загрузили или создали с помощью редактора документы. То в печатной форме по левый край документа будут qr-коды, содержащие в себе информацию о подписантах, а именно: 
     - Название компании, если это юр. лицо, иначе там будет ФИО физ. лица. 
     - ФИО работника или физ лица
     - Должность работника или "Физ. лицо" для физю лиц
     - Дата подписания документа
        ```
            Компания: Тестов Тест Тестулы
            ФИО: Тестов Тест Тестулы
            Должность: Физ. лицо
            Дата подписания: ДД.ММ.ГГГГ ЧЧ:ММ:СС
        ```
        
4) ***Папки с подписью***
   - Папок столько же сколько и подписантов. Названия папок начинается со слова "Подпись" и далее идет ФИО подписантов. В каждой папке находится `.xml` файл.


## Описание подписей

> \*Данный текст не является частью анализа и нужно удалить при сдаче.
> 
> В этом разделе нужно проанализировать результирующие подписи. Целью этого раздела является анализ содержимого подписей, какая информация хранится в подписи, что конкретно подписывается и любая другая полезная информация.
>
> В данной секции обязательно описать что подписывается ЭЦП ключом - хеш, бинарные данные, UUID документ и т.д. Если это хеш, то как он формируется.
>  
>  В данной секции следует описать команды и инструменты используемые для анализа. Если это bash скрипты то написать команду, вывод и описание.
>   
>  Можно прилагать скрины.

## Описание процесса верификации подписей на сервисе

> \*Данный текст не является частью анализа и нужно удалить при сдаче.
> 
> В этом разделе нужно описать процессы того, как сервис предлагает проверить подпись на легитимность. Перечислить все доступные методы рекомендованные сервисом. Возможно данная информация будет храниться в статьях, блогах или FAQ сервиса.
>  
>  Можно прилагать скрины.

## Описание процесса верификации подписей вне сервиса

> \*Данный текст не является частью анализа и нужно удалить при сдаче.
> 
> В этом разделе нужно описать процессы того, как можно проверить подписи на легитимность вне сервиса, используя технические инструменты - kalkan, openssl и т.д.

---

> \*Данный текст не является частью анализа и нужно удалить при сдаче.
> 
> Если у вас есть дополнительные секции, которые вы хотели бы включить, то оформите ее согласно структуре во всем документе.
