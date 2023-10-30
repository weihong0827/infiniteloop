---
{"dg-publish":true,"permalink":"/projects/ascenda-loyalty-point-system/","created":"2023-10-27T15:38:51.876+08:00","updated":"2023-10-31T00:09:58.655+08:00"}
---

  

# Intro

This is a very backend focused project to build an robust API to handle the loyalty program and the exchanging of the customer’s credit points.

I work on this project together with my teammate Varshini, Hayrul, Nicholas and Kang Ming!

## Tech Stack Chosen

- `Golang` with [gin](https://github.com/gin-gonic/gin) frame work for API building
- `Postgres` is our choice of database since `relational database` have a stricter rules in terms of how each fields are defined.
- `Next.js` to build a simple UI to carry out the basic interaction with the backend API
- `Cypress` for frontend testing
- `Golang`’s testing package for backend unit and integration testing

# Features

## Sample Architecture

Here’s a proposed architecture for how you could build the apps to mimic the flows we have here in Ascenda.

![AscendaAritechureDiagram.jpg](/img/user/Projects/attachments/AscendaAritechureDiagram.jpg)

  

TransferConnect (where the core logic of points processing exists)

**Bank App**: Customer facing frontend where a user can submit their points redemption by supplying a loyalty program membership. A simple demo using the different APIs from the TransferConnect App would suffice.

**TransferConnect App**: Backend only app which handles the API requests of points transfers from the banks and collates them for processing with the loyalty programs.

**Loyalty Program**: Fulfills the points to be redeemed in their system by ingesting files you generate and upload to the provisioned SFTP folder; You don’t need to build this; You can mock the process by uploading the handback file you generate and have your TransferConnect app ingest it.

### My thoughts

- The `validation of membership` can be done in the `backend` when the user try to submit a credit request and reject when it is not correct, there is no need for a dedicated endpoint for it
- The `approval` of the `credit requests` can be built using an `admin system` to allow the third party loyal program to `interact with the API` to `update the database` accordingly
    - `Realtime update`, whenever the third party approves, customer will get `notify immediately` instead of waiting for the server to fetch the hand back file and process
    - `Reduce` the potential `human error` when the third party programs create the hand back files
    - It definitely `more work` on both frontend and backend
- A better way to `notify user` will be just through `email`, instead of `webhook` since this is a webapp and not a phone app, most people do not turn on the notification feature on webapp (at least the people i know)

# Planning

Some of our diagrams during the planning phase before we start on the project.

## Use Case diagram

The diagram below illustrates the `interactions` between `all the parties` and the potential `threats` to the webapp and `preventions` that can be taken

![Projects/attachments/Untitled.png|Untitled.png](/img/user/Projects/attachments/Untitled.png)

  

## Software Development Process

### Iterative and Incremental Model

Since the general expected requirements have been outlined by the client, which is Ascenda in this case, we believe an `iterative` and `incremental` model is the `ideal strategy` for this project.

We believe this is the right approach because incremental development allows us to `make adjustments` `early` in the process rather than waiting until the end, and iterative development allows us to `make continuous improvements`. Furthermore, since a `working prototype` may be `produced early` on in the project using this development process, it is possible to `isolate flaws` in functions or designs as it is being `reviewed` and `discussed`.

![Projects/attachments/Untitled 1.png|Untitled 1.png](/img/user/Projects/attachments/Untitled%201.png)

Fig 1. An example of Iterative and Incremental Development Process

Some of the advantages of using this software development process is that progress can be `easily measured`, most problems can be `detected` during iteration and `higher risk` may be `handled` with as an `early priority`, and `functional prototypes` can be built early in the project life cycle. However, this process also comes with a drawback that the need for `more intensive project management` and `strong design` of the `system architecture` may be required. If there are any errors found in the later iterations, then all the code released at the end of the other iterations also has to be rectified as well since there may be little overlap between each iteration.

## Database Design

Since the database of our choice is Postgres, which is a `relational database`, the `relationship` between each `table` should be `well established` before we start on the project, and we create our diagram using [dbdiagram.io](http://dbdiagram.io) which allows us to `visualise` the database relationships base on the `sql code` that we write and export to the relevant sql files

[https://dbdiagram.io/embed/62c25b8d69be0b672c901954](https://dbdiagram.io/embed/62c25b8d69be0b672c901954)

  

### UML Diagram

A very general overview of the implementation the entire app. With relevant fields from the database design and corresponding methods required to pass the

![UML-front__backend.jpg](/img/user/Projects/attachments/UML-front__backend.jpg)

# Backend

  
## SQLC

generate golang database CRUD code using pure sql syntax

- [documentation](https://docs.sqlc.dev/en/stable/howto/update.html)

### Usage

1. Download the `cli` tool
    1. MAC OS: `brew install sqlc`
    2. Ubuntu: `sudo snap install sqlc`
    3. Go: `go install github.com/kyleconroy/sqlc/cmd/sqlc@latest`
    4. More details [here](https://docs.sqlc.dev/en/stable/overview/install.html)
2. Initialise
    
    1. `sqlc init`
    2. `sqlc.yaml` should be generated
    3. update the content
    
```
version: "1"
packages:
  - path: "./pkg/models"
    name: "models"
    engine: "postgresql"
    schema: "./pkg/models/migrations"
    queries: "./pkg/models/query"
    emit_json_tags: true
```
1. Database Migration
    
    Before we start on any coding, the database structure has to be defined in the Postgres database.
    
    - `Download` the sql file from the [[Notion/Website CMS/My projects/Current Projects/Ascenda Loyalty Point System/Ascenda Loyalty Point System\|Notion/Website CMS/My projects/Current Projects/Ascenda Loyalty Point System/Ascenda Loyalty Point System]]
    - Use one of the `database migration` library. We use [golang-migrate](https://github.com/golang-migrate/migrate).
    - copy the content in the sql file that is downloaded to the `000001_init.up.sql`
    - run `migrate -source file://pkg/models/migrations -database "${PSQL_LINK}?sslmode=disable" up`
    - The command can be written into a [[Notion/Website CMS/My projects/Current Projects/Ascenda Loyalty Point System/Ascenda Loyalty Point System\|Notion/Website CMS/My projects/Current Projects/Ascenda Loyalty Point System/Ascenda Loyalty Point System]] and simply be called using `make migrate`.
    - Now open any `Graphical User Interface (GUI)` for Postgres and connect to the database, you should see the tables well defined and ready to use.
4. Write sql queries in the `queries` directory
    
    1. use `loyalty program` as an example below
    2. `database schema` must be defined first in the schema directory
    3. `schema` and `query` are to be defined in different file in the directory specified in the `sqlc.yaml` file
    
```sql
//SCHEMA inside the ./pkg/models/migrations folder
CREATE TABLE "loyalty_program" (
  "id" bigserial PRIMARY KEY,
  "name" varchar NOT NULL,
  "currency_name" varchar NOT NULL,
  "processing_time" varchar NOT NULL,
  "description" varchar,
  "enrollment_link" varchar NOT NULL,
  "terms_condition_link" varchar NOT NULL,
  "format_regex" varchar NOT NULL,
  "partner_code" varchar NOT NULL,
  "initial_earn_rate" float NOT NULL
);
```
    
```sql
//query file loyalty_program.sql inside ./pkg/models/query
-- name: GetLoyaltyByID :one
SELECT * FROM loyalty_program
WHERE id = $1 LIMIT 1;

-- name: GetLoyaltyByName :one
SELECT * FROM loyalty_program
WHERE name = $1 LIMIT 1;

-- name: ListLoyalty :many
SELECT * FROM loyalty_program
ORDER BY name;

-- name: CreateLoyalty :one
INSERT INTO loyalty_program(
  name, currency_name,processing_time,description,enrollment_link,
  terms_condition_link,format_regex,partner_code,initial_earn_rate
) VALUES (
  $1, $2, $3, $4, $5, $6, $7, $8, $9
)
RETURNING *;

-- name: UpdateLoyalty :exec
UPDATE loyalty_program 
SET name = COALESCE($1,name),
currency_name = COALESCE($2,currency_name),
processing_time = COALESCE($3,processing_time),
description = COALESCE($4,description),
enrollment_link = COALESCE($5,enrollment_link),
terms_condition_link = COALESCE($6,terms_condition_link),
format_regex = COALESCE($7,format_regex),
partner_code = COALESCE($8,partner_code),
initial_earn_rate = COALESCE($9,initial_earn_rate)
WHERE id = $10;


-- name: DeleteLoyalty :exec
DELETE FROM loyalty_program
WHERE id = $1;
```
    
5. Generate CRUD
    
    1. `sqlc generate`
    2. `db.go` contains the database transaction and instantiation
    3. `models.go` contains the struct of the data model
    4. `loyalty_program.go` contains the go functions for the relevant operations matching to the sql query
    5. Sample result
    
```go
**const createLoyalty = `-- name: CreateLoyalty :one
INSERT INTO loyalty_program(
  name, currency_name,processing_time,description,enrollment_link,
  terms_condition_link,format_regex,partner_code,initial_earn_rate
) VALUES (
  $1, $2, $3, $4, $5, $6, $7, $8, $9
)
RETURNING id, name, currency_name, processing_time, description, enrollment_link, terms_condition_link, format_regex, partner_code, initial_earn_rate
`

type CreateLoyaltyParams struct {
	Name               string  `json:"name"`
	CurrencyName       string  `json:"currency_name"`
	ProcessingTime     string  `json:"processing_time"`
	Description        string  `json:"description"`
	EnrollmentLink     string  `json:"enrollment_link"`
	TermsConditionLink string  `json:"terms_condition_link"`
	FormatRegex        string  `json:"format_regex"`
	PartnerCode        string  `json:"partner_code"`
	InitialEarnRate    float64 `json:"initial_earn_rate"`
}

func (q *Queries) CreateLoyalty(ctx context.Context, arg CreateLoyaltyParams) (LoyaltyProgram, error) {
	row := q.db.QueryRowContext(ctx, createLoyalty,
		arg.Name,
		arg.CurrencyName,
		arg.ProcessingTime,
		arg.Description,
		arg.EnrollmentLink,
		arg.TermsConditionLink,
		arg.FormatRegex,
		arg.PartnerCode,
		arg.InitialEarnRate,
	)
	var i LoyaltyProgram
	err := row.Scan(
		&i.ID,
		&i.Name,
		&i.CurrencyName,
		&i.ProcessingTime,
		&i.Description,
		&i.EnrollmentLink,
		&i.TermsConditionLink,
		&i.FormatRegex,
		&i.PartnerCode,
		&i.InitialEarnRate,
	)
	return i, err
}**
```
    

### Problems

1. `sql.Null<type>` is used for defining of fields that can contain `NULL`
    1. It has the following struct
	```go
type NullString struct {
	Stringstring
	Validbool // Valid is true if String is not NULL
}
	```
    2. this struct is very problematic when is used by the `gin framework` to `marshal` and `unmarshal` the json data come of the api request body.
    3. Some of the [solutions](https://stackoverflow.com/questions/51961358/how-do-you-marshal-a-sql-nullstring-such-that-the-output-is-flattened-to-give-ju) that is found online are to define a wrapper class for this type and define a `MarshalJSON` function to define the `marshal` behaviour. Then put the wrapper class in the struct that needs these `Null Fields`
    4. The problem is, the struct that uses these Null Fields is `auto generated` using `sqlc` using the `sql queries` defined.
    5. It can be changed to the new wrapper class, but every there is a need to `update the queries` and `sqlc generate` is run to generate the code, the new code will overwrite the changes that you have made.
    6. The work around is to define null fields as `NOT NULL` but with a `DEFAULT` to an empty string.

## Reward Calculation

This is one of the core function of the app. There is a bonus requirement from Ascenda to make the reward varies base on promotions given by third party. Hence the logic have to be well thought out and carefully implemented.

`Promotion` table is defined with the following fields
```sql
CREATE TABLE "promotion" (
  "id" bigserial PRIMARY KEY,
  "program" int NOT NULL,
  "promo_type" promo_type_enum NOT NULL,
  "start_date" date NOT NULL,
  "end_date" date NOT NULL,
  "earn_rate_type" earn_rate_type_enum NOT NULL,
  "constant" float NOT NULL,
  "card_tier" int,
  "loyalty_membership" int
);
```

- `start` and `end dates` (to specify the `period` of promotion)
- `program` which is a `foreign key` points to which `loyalty program` that this promotion is related with
- `promot_type` whether it is `onetime` (each customer can only claim once) or `ongoing` (valid for as long as the current date of making credit transfer falls inside ) promotion
- `earn_rate_type` whether the additional reward is granted using `addition` or `multiplication`
- `constant` the `amount` to multiply or add
- `card_tier` a `foreign key` points to a predefined card tier by the bank. Eg. Gold, Platinum… This field is `optional`, if provided, only user with that particular card tier will be able to enjoy the promotion
- `loyalty_membership` a `foreign key` points to a predefined card tier by the third party loyalty program. Eg. Gold, Platinum… This field is `optional`, if provided, only user with that particular membership will be able to enjoy the promotion. We later found that this field is `not practical` since the membership details is only available to the third party and we do not have access

### Different Scenarios

The rewards varies under different conditions. Test cases are written for each of the condition.

Some of the variables used `creditToTransfer` — the amount of credit that the user plan to transfer. `initalEarnRate` the earn rate defined by the third party loyalty program

  

- Reward without promotion
    - reward = creditToTransfer * initalEarnRate
- Reward with on onging promotion
    - reward = creditToTransfer * initalEarnRate (+/*) constant
- Reward with on onetime promotion
    - Check the past credit to see if this user has any active credit transfer use the current promo
    - if no
        - reward = creditToTransfer * initalEarnRate (+/*) constant
    - if yes
        - reward = creditToTransfer * initalEarnRate
- Reward with a card tier specified
    - Check the user has the matching card tier
    - if yes
        - reward = creditToTransfer * initalEarnRate (+/*) constant
    - if no
        - reward = creditToTransfer * initalEarnRate
- Reward with multiple promotions that is valid
    - calculate all the reward that the user might get and return only the highest one

  

### Code

The main function for the above requirements.

```go
func CalculateReward(c context.Context, query *models.Queries, body models.TransferParams) (result float64, promoUsed sql.NullInt32, err error) {
	if body.CreditToTransfer < 0 {
		// return 0,error
	}
	program, err := query.GetLoyaltyByID(c, int64(body.ProgramId))
	if err != nil {
		return 0, sql.NullInt32{Valid: false}, nil
	}

	getPromoParam := models.GetPromotionByDateRangeParams{
		Column1: time.Now().Format("2006-01-02"),
		Program: int32(program.ID),
	}

	promotions, err := query.GetPromotionByDateRange(c, getPromoParam)
	if err != nil {
		return 0, sql.NullInt32{Valid: false}, nil
	}

	user, err := query.GetUserByID(c, int64(body.UserId))
	if err != nil {
		return 0, sql.NullInt32{Valid: false}, nil
	}
	var base float64 = program.InitialEarnRate * body.CreditToTransfer
	var promoIdUsed int32 = 0
	var max float64 = base
	for _, promotion := range promotions {
		var tempReward float64 = 0
		if promotion.PromoType == "onetime" {
			args := models.GetCreditRequestByPromoParams{
				Program:   int32(program.ID),
				PromoUsed: sql.NullInt32{Valid: true, Int32: int32(promotion.ID)},
			}
			_, err = query.GetCreditRequestByPromo(c, args)

			//skip the loop if there is result found
			// if err.Error()!="sql: no rows in result set"{
			if err != sql.ErrNoRows {
				fmt.Println("no pass request made")
				continue
			}
		}

		if promotion.CardTier.Valid && user.CardTier.Valid {
			if promotion.CardTier.Int32 == user.CardTier.Int32 {
				tempReward = processReward(promotion, base)
			}
		} else if promotion.CardTier.Valid {
			continue

		} else {
			tempReward = processReward(promotion, base)

		}

		if tempReward != 0 {
			if tempReward > max {
				max = tempReward
				promoIdUsed = int32(promotion.ID)
			}
		}
	}
	if promoIdUsed != 0 {
		return max, sql.NullInt32{Int32: promoIdUsed, Valid: true}, nil
	} else {
		return max, sql.NullInt32{Valid: false}, nil
	}

}
```

### Database Transaction

To make sure that the `right amount` is deducted or credited to the user, a `series` of `database operation` has to be carried out. If `any` of the operation `failed`. The entire series operation operation should be `abort` and revert to the previous state.

An example: bank transfer

- Alice want to transfer 1000 dollar to Bob
- Right flow
    - Deduct 1000 dollar from Alice’s account
    - Credit 1000 dollar to Bob’s account
- if the database `fails` to deduct 1000 dollar from Alice’s account. It should `abort` entire flow and `not credit` the amount to bob’s account

Here comes the idea of [`**Transaction**`](https://en.wikipedia.org/wiki/Database_transaction)

### Code to implement transaction

The deduct of credit from the user’s credit balance should only be carried out if a new credit request is created successfully. if the deduction is not successful, it should revert the process

```go
func (store *Store) execTx(ctx context.Context,fn func(*Queries) error) error{
  tx,err := store.db.BeginTx(ctx,nil)
  if err!=nil{
    return err
  }
  q := New(tx)
  err = fn(q)
  if err!=nil{
    if rbErr:=tx.Rollback(); rbErr!=nil{
      return fmt.Errorf("tx err: %v, rb err: %v",err,rbErr)
    }
    return err
  }
  return tx.Commit()
}

func (store *Store) CreditTransferOut(ctx context.Context,arg TransferParams,promo sql.NullInt32) (CreditRequest,error){
  var result CreditRequest
  err:= store.execTx(ctx, func (q *Queries)error{
    var err error
    request:=CreateCreditRequestParams{
      UserID:arg.UserId,
      Program:arg.ProgramId,
      MemberID:arg.MembershipId,
      CreditUsed:arg.CreditToTransfer,
      RewardShouldReceive:arg.RewardShouldReceive,
      PromoUsed:promo,
      TransactionTime:sql.NullTime{Time:time.Now(),Valid:true},
      TransactionStatus:TransactionStatusEnum("created"),
    }
    result,err =q.CreateCreditRequest(
     ctx,request, 
    )
    if err!=nil{
      return err
    }
    balanceParam := DecreBalanceParams {
      CreditBalance:arg.CreditToTransfer,
      ID: int64(arg.UserId),
    }

    //TODO prevent deadlock
    err = q.DecreBalance(ctx,balanceParam)
    if err!=nil{
      return err
    }
    return nil
      
  })
  return result,err 

}
```

  

### Database Dead Lock Prevention

Dead lock happens when there is not resources to be allocated to current ongoing processes to let them finish and release the resources that they are holding.

# Testing

Extensive testing is done with a plenty of test cases written for unit and integration testing on the backend and frontend testing is done using cypress

## Code Snippets

Some of the test cases that we wrote

### Unit testing for database operations

```go
func createLoyaltyObject() CreateLoyaltyParams {
	arg := CreateLoyaltyParams{
		Name:               utils.RandomString(6),
		CurrencyName:       utils.RandomString(6),
		ProcessingTime:     utils.RandomString(4),
		Description:        sql.NullString{String: utils.RandomString(20), Valid: true},
		EnrollmentLink:     utils.RandomString(20),
		TermsConditionLink: utils.RandomString(20),
		FormatRegex:        utils.RandomString(10),
		PartnerCode:        utils.RandomString(5),
		InitialEarnRate:    utils.RandomFloat(10),
	}
	return arg
}
func TestCreateLoyalty(t *testing.T) {
	obj := createLoyaltyObject()
	program, err := testQueries.CreateLoyalty(context.Background(), obj)
	require.NoError(t, err)
	require.NotEmpty(t, program)

	require.Equal(t, program.Name, obj.Name)
	require.Equal(t, program.CurrencyName, obj.CurrencyName)
	require.Equal(t, program.ProcessingTime, obj.ProcessingTime)
	require.Equal(t, program.Description.String, obj.Description.String)
	require.Equal(t, program.EnrollmentLink, obj.EnrollmentLink)
	require.Equal(t, program.TermsConditionLink, obj.TermsConditionLink)
	require.Equal(t, program.FormatRegex, obj.FormatRegex)
	require.Equal(t, program.PartnerCode, obj.PartnerCode)
	require.Equal(t, program.InitialEarnRate, obj.InitialEarnRate)
	require.NotZero(t, program.ID)
}
func TestDeleteLoyalty(t *testing.T) {
	obj := createLoyaltyObject()
	program, err := testQueries.CreateLoyalty(context.Background(), obj)
	require.NoError(t, err)
	require.NotEmpty(t, program)

	deleteErr := testQueries.DeleteLoyalty(context.Background(), program.ID)
	require.NoError(t, deleteErr)

	_, getErr := testQueries.GetLoyaltyByID(context.Background(), program.ID)
	require.EqualError(t, getErr, "sql: no rows in result set")
}
```

### Integration testing on reward calculation

```go
// when promo ask for cartier but use no cardtier/ user's cardtier is below what is requested
func TestRewardCalPromoOutOfRange(t *testing.T) {

	cardTierArgs := models.CreateCardTierParams{
		Name: utils.RandomString(7),
		Tier: 2,
	}
	cardTier, err := testQueries.CreateCardTier(context.Background(), cardTierArgs)
	createLoyaltyArgs := createLoyaltyObject()
	createUserArgs := createUserObject(sql.NullInt32{Valid: false})
	var creditToTransfer float64 = 100
	program, err := testQueries.CreateLoyalty(context.Background(), createLoyaltyArgs)
	require.NoError(t, err)
	user, err := testQueries.CreateUser(context.Background(), createUserArgs)
	require.NoError(t, err)
	args := models.TransferParams{
		UserId:           int32(user.ID),
		ProgramId:        int32(program.ID),
		CreditToTransfer: float64(creditToTransfer),
		MembershipId:     utils.RandomString(6),
	}
	startDate, err := time.Parse("2006-01-02", "2022-07-01")
	require.NoError(t, err)
	endDate, err := time.Parse("2006-01-02", "2022-07-15")
	require.NoError(t, err)

	var constant float64 = 1000
	createPromoArgs := models.CreatePromotionParams{
		Program:      int32(program.ID),
		PromoType:    models.PromoTypeEnum("ongoing"),
		StartDate:    startDate,
		EndDate:      endDate,
		EarnRateType: models.EarnRateTypeEnum("add"),
		Constant:     float64(constant),
		CardTier:     sql.NullInt32{Valid: true, Int32: int32(cardTier.ID)},
	}
	_, err = testQueries.CreatePromotion(context.Background(), createPromoArgs)
	require.NoError(t, err)
	result, _, err := CalculateReward(context.Background(), testQueries, args)
	require.NoError(t, err)
	expected := creditToTransfer * (program.InitialEarnRate)
	require.Equal(t, expected, result)
}
```

## Makefile

shorten some of the commands

Simply run `make <command>` for the shortened command that is specified below

```shell
include .env
export
docker-postgres:
	docker run --name postgres-db -p 55001:5432 -e POSTGRES_PASSWORD=postgrespw -d postgres
migrations:
	@read -p "Enter the name of the migration: " migration_name;\
	migrate create -ext sql -dir pkg/models/migrations -seq $$migration_name
migrate:
	migrate -source file://pkg/models/migrations -database "${PSQL_LINK}" up 

migrate-down:
	migrate -source file://pkg/models/migrations -database "${PSQL_LINK}" down 

test-model:
	go test esc/ascendaRoyaltyPoint/pkg/models -v -cover

test-controllers:
	go test esc/ascendaRoyaltyPoint/pkg/controllers -v -cover

sqlc:
	sqlc generate
```

  

# Conclusion

This project is currently at its final stage before submission and I think that there are still many more features and use case scenarios that can be implemented but could not due to the limited amount of time.

Nevertheless, I have horned my skills in making database design and interaction with relational database. I also picked up the basic syntax and concepts of golang which has been one of my biggest skill that i want to learn on my ‘wish list’.

One of the most important lesson or skill that I took away is how to construct promoter test cases to make sure that the code runs with less error.

Even though there are curses like:

> _Testing can only find the presence of errors, not their absence —_ **Edsger W. Dijkstra**
> 
> _Verification can only find the absence of errors, not their presence (in general) —_ **Andreas Zeller**

But we should still do extensive testing on our program, because any bugs may cause significant decrease in user experience or bring detrimental impact to the business. Even Microsoft spent 75% of the time testing

Every new project, all new experience. With different tools that I get to explore for different feature implementation, makes the world of programming full of possibility!