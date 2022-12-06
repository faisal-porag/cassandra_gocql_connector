# cassandra_gocql_connector



```sh 

package main

import (
	"fmt"
	"github.com/gocql/gocql"
	"log"
	"time"
)

//
//type CassandraConfig struct {
//	host        string
//	port        string
//	keyspace    string
//	consistency string
//}
//
//var cassandraConfig = CassandraConfig{
//	host:        getEnv("CASSANDRA_HOST", "127.0.0.1"),
//	port:        getEnv("CASSANDRA_PORT", "9042"),
//	keyspace:    getEnv("CASSANDRA_KEYSPACE", "test"),
//	consistency: getEnv("CASSANDRA_CONSISTENCY", "LOCAL_QUORUM"),
//}
//
//func getEnv(key, fallback string) string {
//	if value, ok := os.LookupEnv(key); ok {
//		return value
//	}
//
//	return fallback
//}

var ScyllaConnection *gocql.Session

//func initSession() {
//	port := func(p string) int {
//		i, err := strconv.Atoi(p)
//		if err != nil {
//			return 9042
//		}
//
//		return i
//	}
//
//	consistency := func(c string) gocql.Consistency {
//		gc, err := gocql.MustParseConsistency(c)
//		if err != nil {
//			return gocql.All
//		}
//
//		return gc
//	}
//
//	cluster := gocql.NewCluster(cassandraConfig.host)
//	cluster.Port = port(cassandraConfig.port)
//	cluster.Keyspace = cassandraConfig.keyspace
//	cluster.Consistency = consistency(cassandraConfig.consistency)
//
//	log.Print(cluster.WriteTimeout)
//	log.Print(cluster.Timeout)
//	log.Print(cluster.ConnectTimeout)
//
//	s, err := cluster.CreateSession()
//	if err != nil {
//		log.Printf("ERROR: fail create cassandra session, %s", err.Error())
//		panic(err)
//	} else {
//		log.Print("Scylla db connection successful")
//	}
//	Session = s
//}


type PersonalData struct {
	Id string
	Name  string
	Email  string
	Address  string
	Friends     []string
}


func ScyllaConnectionSetup() {
	cluster := gocql.NewCluster("127.0.0.1")
	cluster.Consistency = gocql.Quorum
	cluster.Keyspace = "test"
	cluster.Authenticator = gocql.PasswordAuthenticator{
		Username: "cassandra",
		Password: "cassandra",
	}
	cluster.ConnectTimeout = time.Second * 2
	cluster.WriteTimeout = time.Second * 1

	log.Println("cluster.Timeout =", cluster.Timeout)
	log.Println("cluster.WriteTimeout =", cluster.WriteTimeout)
	log.Println("cluster.ConnectTimeout =", cluster.ConnectTimeout)

	S, err := cluster.CreateSession()
	if err != nil {
		log.Println("ScyllaConnection connect-ERROR: ", err)
		panic(err)
	} else {
		log.Println("ScyllaConnection connected")
	}
	ScyllaConnection = S
}

func clearSession() {
	ScyllaConnection.Close()
}

func GetData(id string, scylla *gocql.Session) []PersonalData {
	var personalData []PersonalData
	m := map[string]interface{}{}

	iter := scylla.Query("SELECT * FROM personal_data WHERE id=?", id).Iter()
	for iter.MapScan(m) {
		personalData = append(personalData, PersonalData{
			Id:      m["id"].(string),
			Name:            m["name"].(string),
			Email:        m["email"].(string),
			Address:       m["address"].(string),
			Friends:           m["friends"].([]string),
		})
		m = map[string]interface{}{}
	}
	return personalData
}

func main()  {
	log.Print("Scylla R&D")

	ScyllaConnectionSetup()

	data := GetData("9ee85e9e-7f56-4893-8de3-68e5f19a023f", ScyllaConnection)

	fmt.Println(data[0].Id)
	fmt.Println(data[0].Name)
	fmt.Println(data[0].Email)
	fmt.Println(data[0].Address)

	clearSession()
}


```
