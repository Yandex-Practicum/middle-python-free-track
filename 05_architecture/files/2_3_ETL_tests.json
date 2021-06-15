{
	"info": {
		"_postman_id": "d817fc79-dd7b-4419-98ad-793ee72e2c87",
		"name": "ETLTests",
		"schema": "https://schema.getpostman.com/json/collection/v2.1.0/collection.json"
	},
	"item": [
		{
			"name": "Проверка количества элементов",
			"event": [
				{
					"listen": "test",
					"script": {
						"id": "487c7688-e2c7-462f-aa05-85a5060f4077",
						"exec": [
							"pm.test(\"Success answer\", function() {",
							"    pm.response.to.have.status(200);",
							"});",
							"",
							"pm.test(\"Compare number of records\", function() {",
							"    var jsonData = pm.response.json();",
							"    pm.expect(jsonData['hits']['total']['value']).to.equal(999);",
							"});"
						],
						"type": "text/javascript"
					}
				}
			],
			"request": {
				"method": "GET",
				"header": [],
				"url": {
					"raw": "{{base_url}}/movies/_search",
					"host": [
						"{{base_url}}"
					],
					"path": [
						"movies",
						"_search"
					]
				}
			},
			"response": []
		},
		{
			"name": "Запрос на поиск N/A элементов",
			"event": [
				{
					"listen": "test",
					"script": {
						"id": "7349e669-3968-4683-b598-ce558389cccb",
						"exec": [
							"pm.test(\"Success answer\", function() {",
							"    pm.response.to.have.status(200);",
							"});",
							"",
							"pm.test(\"Search for N/A records\", function() {",
							"    var jsonData = pm.response.json();",
							"    pm.expect(jsonData['hits']['total']['value']).to.equal(7);",
							"    pm.expect(pm.response.text()).not.to.have.string('N/A');",
							"});"
						],
						"type": "text/javascript"
					}
				}
			],
			"protocolProfileBehavior": {
				"disableBodyPruning": true
			},
			"request": {
				"method": "GET",
				"header": [],
				"body": {
					"mode": "raw",
					"raw": "{\n    \"query\": {\n        \"query_string\": {\n            \"query\": \"N//A\"\n        }\n    }\n}",
					"options": {
						"raw": {
							"language": "json"
						}
					}
				},
				"url": {
					"raw": "{{base_url}}/movies/_search",
					"host": [
						"{{base_url}}"
					],
					"path": [
						"movies",
						"_search"
					]
				}
			},
			"response": []
		},
		{
			"name": "Запрос на поиск данных по слову camp",
			"event": [
				{
					"listen": "test",
					"script": {
						"id": "0253364a-1751-4082-b2d4-83cdf906713f",
						"exec": [
							"pm.test(\"Success answer\", function() {",
							"    pm.response.to.have.status(200);",
							"});",
							"",
							"pm.test(\"Compare number of records\", function() {",
							"    var jsonData = pm.response.json();",
							"    pm.expect(jsonData['hits']['total']['value']).to.equal(24);",
							"    pm.expect(jsonData['hits']['max_score']).to.equal(7.500733);",
							"});"
						],
						"type": "text/javascript"
					}
				}
			],
			"protocolProfileBehavior": {
				"disableBodyPruning": true
			},
			"request": {
				"method": "GET",
				"header": [],
				"body": {
					"mode": "raw",
					"raw": "{\n    \"query\": {\n        \"multi_match\": {\n            \"query\": \"camp\",\n            \"fuzziness\": \"auto\",\n            \"fields\": [\n                \"actors_names\",\n                \"writers_names\",\n                \"title\",\n                \"description\",\n                \"genre\"\n            ]\n        }\n    }\n}",
					"options": {
						"raw": {
							"language": "json"
						}
					}
				},
				"url": {
					"raw": "{{base_url}}/movies/_search",
					"host": [
						"{{base_url}}"
					],
					"path": [
						"movies",
						"_search"
					]
				}
			},
			"response": []
		},
		{
			"name": "Запрос данных по актеру",
			"event": [
				{
					"listen": "test",
					"script": {
						"id": "43502fc5-5678-41e2-b48e-6295924304b0",
						"exec": [
							"pm.test(\"Success answer\", function() {",
							"    pm.response.to.have.status(200);",
							"});",
							"",
							"pm.test(\"Compare number of records\", function() {",
							"    var jsonData = pm.response.json();",
							"    pm.expect(jsonData['hits']['total']['value']).to.equal(6);",
							"});"
						],
						"type": "text/javascript"
					}
				}
			],
			"protocolProfileBehavior": {
				"disableBodyPruning": true
			},
			"request": {
				"method": "GET",
				"header": [],
				"body": {
					"mode": "raw",
					"raw": "{\n    \"query\": {\n        \"nested\": {\n            \"path\": \"actors\",\n            \"query\": {\n                \"bool\": {\n                    \"must\": [\n                        {\n                            \"match\": {\n                                \"actors.name\": \"Greg Camp\"\n                            }\n                        }\n                    ]\n                }\n            }\n        }\n    }\n}",
					"options": {
						"raw": {
							"language": "json"
						}
					}
				},
				"url": {
					"raw": "{{base_url}}/movies/_search",
					"host": [
						"{{base_url}}"
					],
					"path": [
						"movies",
						"_search"
					]
				}
			},
			"response": []
		},
		{
			"name": "Запрос данных с дублями полей",
			"event": [
				{
					"listen": "test",
					"script": {
						"id": "7fb8cad0-5b19-48b8-8d89-20ffec8b6042",
						"exec": [
							"pm.test(\"Success answer\", function() {",
							"    pm.response.to.have.status(200);",
							"});",
							"",
							"pm.test(\"Compare writers_names\", function() {",
							"    var jsonData = pm.response.json();",
							"    pm.expect(jsonData['hits']['total']['value']).to.equal(1);",
							"    pm.expect(jsonData['hits']['hits'][0]['_source']['writers_names']).to.eql([\"Lucien Hubbard\"]);",
							"});"
						],
						"type": "text/javascript"
					}
				}
			],
			"protocolProfileBehavior": {
				"disableBodyPruning": true
			},
			"request": {
				"method": "GET",
				"header": [],
				"body": {
					"mode": "raw",
					"raw": "{\n    \"query\": {\n        \"term\": {\n            \"id\": {\n                \"value\": \"tt0022430\"\n            }\n        }\n    }\n}",
					"options": {
						"raw": {
							"language": "json"
						}
					}
				},
				"url": {
					"raw": "{{base_url}}/movies/_search",
					"host": [
						"{{base_url}}"
					],
					"path": [
						"movies",
						"_search"
					]
				}
			},
			"response": []
		},
		{
			"name": "Запрос данных с одним сценаристом",
			"event": [
				{
					"listen": "test",
					"script": {
						"id": "fc639d7a-2a70-45a3-8488-ec309bd1db84",
						"exec": [
							"pm.test(\"Success answer\", function() {",
							"    pm.response.to.have.status(200);",
							"});",
							"",
							"pm.test(\"Compare writers_names\", function() {",
							"    var jsonData = pm.response.json();",
							"    pm.expect(jsonData['hits']['total']['value']).to.equal(1);",
							"    pm.expect(jsonData['hits']['hits'][0]['_source']['writers_names']).to.eql([\"Craig Hutchinson\"]);",
							"});"
						],
						"type": "text/javascript"
					}
				}
			],
			"protocolProfileBehavior": {
				"disableBodyPruning": true
			},
			"request": {
				"method": "GET",
				"header": [],
				"body": {
					"mode": "raw",
					"raw": "{\n    \"query\": {\n        \"term\": {\n            \"id\": {\n                \"value\": \"tt0004637\"\n            }\n        }\n    }\n}",
					"options": {
						"raw": {
							"language": "json"
						}
					}
				},
				"url": {
					"raw": "{{base_url}}/movies/_search",
					"host": [
						"{{base_url}}"
					],
					"path": [
						"movies",
						"_search"
					]
				}
			},
			"response": []
		},
		{
			"name": "Запрос данных без режиссера",
			"event": [
				{
					"listen": "test",
					"script": {
						"id": "bba7846d-0a26-4eba-98e3-e0ea35ed2587",
						"exec": [
							"pm.test(\"Success answer\", function() {",
							"    pm.response.to.have.status(200);",
							"});",
							"",
							"pm.test(\"Compare director\", function() {",
							"    var jsonData = pm.response.json();",
							"    pm.expect(jsonData['hits']['total']['value']).to.equal(1);",
							"    pm.expect(jsonData['hits']['hits'][0]['_source']['director']).to.be.null;",
							"});"
						],
						"type": "text/javascript"
					}
				}
			],
			"protocolProfileBehavior": {
				"disableBodyPruning": true
			},
			"request": {
				"method": "GET",
				"header": [],
				"body": {
					"mode": "raw",
					"raw": "{\n    \"query\": {\n        \"term\": {\n            \"id\": {\n                \"value\": \"tt0006062\"\n            }\n        }\n    }\n}",
					"options": {
						"raw": {
							"language": "json"
						}
					}
				},
				"url": {
					"raw": "{{base_url}}/movies/_search",
					"host": [
						"{{base_url}}"
					],
					"path": [
						"movies",
						"_search"
					]
				}
			},
			"response": []
		}
	],
	"event": [
		{
			"listen": "prerequest",
			"script": {
				"id": "33e2d65f-2a09-4029-8ba3-5b262b5070c0",
				"type": "text/javascript",
				"exec": [
					""
				]
			}
		},
		{
			"listen": "test",
			"script": {
				"id": "a0fe5648-3d9c-46ec-924d-341c500dbd48",
				"type": "text/javascript",
				"exec": [
					""
				]
			}
		}
	],
	"variable": [
		{
			"id": "62d578fe-8ac6-4d2c-9910-0266fd765db0",
			"key": "base_url",
			"value": "http://localhost:9200",
			"type": "string"
		}
	],
	"protocolProfileBehavior": {}
}