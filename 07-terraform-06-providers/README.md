# Домашнее задание к занятию "7.6. Написание собственных провайдеров для Terraform."

## Проценко Анастасия

Бывает, что 
* общедоступная документация по терраформ ресурсам не всегда достоверна,
* в документации не хватает каких-нибудь правил валидации или неточно описаны параметры,
* понадобиться использовать провайдер без официальной документации,
* может возникнуть необходимость написать свой провайдер для системы используемой в ваших проектах.   

## Задача 1. 
Давайте потренируемся читать исходный код AWS провайдера, который можно склонировать от сюда: 
[https://github.com/hashicorp/terraform-provider-aws.git](https://github.com/hashicorp/terraform-provider-aws.git).
Просто найдите нужные ресурсы в исходном коде и ответы на вопросы станут понятны.  


1. Найдите, где перечислены все доступные `resource` и `data_source`, приложите ссылку на эти строки в коде на 
гитхабе.   

* `resource` - https://github.com/hashicorp/terraform-provider-aws/blob/main/internal/provider/provider.go#L902

* `data_source` - https://github.com/hashicorp/terraform-provider-aws/blob/main/internal/provider/provider.go#L423



2. Для создания очереди сообщений SQS используется ресурс `aws_sqs_queue` у которого есть параметр `name`. 
  * С каким другим параметром конфликтует `name`? Приложите строчку кода, в которой это указано. 
```go
ConflictsWith: []string{"name_prefix"}
```
https://github.com/hashicorp/terraform-provider-aws/blob/main/internal/service/sqs/queue.go#L87

   * Какая максимальная длина имени? 

```go
	errors = append(errors, fmt.Errorf("%q cannot be longer than 80 characters", k))
```
https://github.com/hashicorp/terraform-provider-aws/blob/main/internal/service/sfn/validate.go#L11

   * Какому регулярному выражению должно подчиняться имя? 
```go
	if !regexp.MustCompile(`^[a-zA-Z0-9-_]+$`).MatchString(value) {
		errors = append(errors, fmt.Errorf(
			"%q must be composed with only these characters [a-zA-Z0-9-_]: %v", k, value))
```
https://github.com/hashicorp/terraform-provider-aws/blob/main/internal/service/sfn/validate.go#L16
