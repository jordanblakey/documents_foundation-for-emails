Section descriptions

Decscribes what is expected in POST to backend at the end of each section
Note all answers with no alternatives are simple strings
Note UI presentation order is important and should follow order specified below
Normal return from the POST is a 200 response code with exceptions noted below

Affordability Section
	Answers:
		"affordability": "2","10/20"

Personal Section
	This section normal termination is following last answer, but knockout
	tests can force premature termination. Backend discovers permature
	termination based on absence of answer key (i.e. if "gender" is missing then
	"birthdate was a knockout")
	Answers:
		"firstName"
		"middleName"
		"lastName"
		"birthdate"
		"gender": "male","female"
		"heightFeet"
		"heightInches"
		"weight" 
		"street": note this is currently the full address
		"zipcode"
		"email"
		"phone"
		"ssNum"

Personal_Additional_2T
	Answers:
		"citizen": "yes","no"
		"medicalCondition": "yes","no"
		"tobacco": "yes","no"
		"occupation": "employed","student","homemaker","unemployed"
		"currentInsValue"

Replacement
	Answers:
		"replacement": string number representing thousands
		"replacement_terminating": "yes","no"
		"replacement_policies": "yes","no"
		"replacements": array of replacement policies, each entry containing:
			"insurer_name"
			"policy_number"
			"insured_name"
			"replacement_type": "F","R"   F -> user for financing, R -> replacing
		"signature": "yes","no"

Beneficiary
	Answers:
		"beneficiaries": array of beneficiaries, each entry containing:
			"name"
			"percentage": backend expects percentages to total 100

Policy_Final
	Answers:
		"token": Stripe account token
		"signature": "yes","no"

-------------------------

Additional Notes Related to the 10T metadata extension

** Before we are at the stage where we have decided the product (completion of the Affordability section), the "section_config.json" file is assumed to be the default and thus the default workflow is for the 2T

** There was a basic, controlling assumption that we wanted to minimize the back and forth between the fronted/backend and thus built enough programmabiity into the metadata to allow the frontend to get through the 10T workflow without any server posts until some logical endpoint has been reached (either an ineligible, deny, or acceptance)

** The current metadata structure contains very little that is direction to the frontend related to presentation. In its current form, it is to represent the flow through the questions, the basic structure of the associated answers, when we have reached a logical endpoint, and how question sequences are related to each other as necessary.

** There are certain assumed default behaviors on the part of the frontend as it processes the sequences. One default is that lacking any other information, the frontend always proceeds to the next question. So, if it encounters a simple structure like this:

			{
				"type": "input",
				"subType": "radiobox",
				"ref": "tobacco",
				"label": "In the last 12 months have you used any tobacco or nicotine products? Includes cigarettes, e-cigarettes, chewing tobacco, pipes, nicotine patch/gum, vaping or more than24 cigars in that period?"
			}

This is really only a directive to present a yes/no choice (which is the default when no specific choices are given) and to place the answer as "yes" or "no" in the answer data structure for the field "tobacco" -- and then move on to the next question in the sequence.

This can be extended slightly with a structure like this:

			{
				"type": "input",
				"subType": "radiobox",
				"ref": "declinedIns",
				"label": "In the past 5 years, have you been declined for life insurance?",
				"position": "newLine",
				"valueComparison": {
					"value": "yes",
					"action": "submit"
				}
			}

That adds the request for the frontend to compare the answer given to "yes", such that it "yes" is the answer then "submit" (post) back to the server indicating that the frontend is finished and is awaiting its next instructions/

This idea can be used a slightly different way as exemplified by:

			{
				"type": "select",
				"defaultValue": "no",
				"values": [
					{
						"value": "no",
						"text": "No"
					},
					{
						"value": "yes",
						"text": "Yes"
					}
				],
				"ref": "drink",
				"label": "Do you currently drink alcohol?",
				"position": "newLine",
				"dependence": {
					"alcohol": "yes"
				},
				"valueComparison": {
					"value": "no",
					"action": "diabetesGender"
				}
			},

where in this case the valueComparison is used to indicate to the frontend to move directly to the question in the sequence whose "ref" field is called "diabetesGender". This is showing how you can avoid the default behavior of moving to the next question in favor of the behavior of moving to a specific question. This is used in several ways to handle the complexity of the 1.x reflexives. Note also that it exemplifies the declaration that this question depends on the answer to a previous question. It indicates that this question should be processed only if the previous question whose "ref" field is "alcohol" was answered with a "yes".

** There was a recent shift towards using more instances of choices presented with multiple buttons rather than dropdowns in the UI docs, but the representation of them is similar.

			{
				"type": "select",
				"dependence": {
					"drink": "yes"
				},
				"label": "How many drinks do you typically consume per day",
				"defaultValue": "1",
				"values": [
					{
						"value": "0",
						"text": "0"
					},
					{
						"value": "1",
						"text": "1"
					},
					{
						"value": "2",
						"text": "2"
					},
					{
						"value": "3",
						"text": "3"
					},
					{
						"value": "4",
						"text": "4"
					},
					{
						"value": "5",
						"text": "5+"
					}
				],
				"valueComparison":[
					{
						"value": "4",
						"action": "submit"
					},
					{
						"value": "5",
						"action": "submit"
					}
				]
			}

Note here that the "valueComparison" field can be an array of entries (I maintained the ability to have a single dictionary for compatibity with the 2T and older workflows, but we can change it where they are all arrays if we want). At present the frontend would have to support both simple dictionaries and arrays. Again, the default is to always go on to the next question, so here it makes an exception of the "4" and "5" answers requiring the frontend to submit back to the server for those values.

There are quite a few instances in the 10T workflow that use a variation of this approach for handling questions with no/yes/unknown responses:

			{
				"type": "select",
				"ref": "kidneyDisease",
				"label": "In the past two years, have you been medically diagnosed or medically treated for kidney disease (including protein or microalbumin in the urine) or any type of skin ulcers?",
				"position": "newLine",
				"defaultValue": "no",
				"values": [
					{
						"value": "no",
						"text": "No"
					},
					{
						"value": "yes",
						"text": "Yes"
					},
					{
						"value": "unknown",
						"text": "Unknown"
					}
				],
				"valueComparison": [{
					"value": "yes",
					"action": "submit"
				},{"value": "unknown",
				   "action": "submit"}]
			}

where typically we submit back to the server for yes or unknown answers.

A slightly more extreme example:

			{
				"type": "select",
				"ref": "ageDiabetesDiagnosed",
				"label": "How old were you when diabetes was diagnosed?",
				"defaultValue": "",
				"values": [
					{
						"value": "unknown",
						"text": "unknown"
					},
					{
						"value": "0",
						"text": "0 - 20"
					},
					{
						"value": "1",
						"text": "21 - 30"
					},
					{
						"value": "2",
						"text": "31 - 40"
					},
					{
						"value": "3",
						"text": "41 - 50"
					},
					{
						"value": "4",
						"text": "51 - 60"
					}
				],
				"valueComparison": [
				   {
					"value": "unknown",
					"action": "submit"
				   },
				   {
					"value": "0",
					"action": "submit"
				   },
				   {
					"value": "1",
					"action": "submit"
				   },
				   {
					"value": "2",
					"action": "submit"
				   }
				]
			},
			{
				"type": "input",
				"subType": "radiobox",
				"ref": "alcoholAbuseTreatment",
				"label": "in the past 10 years, have you been treated or advised to get treatment by a medical professional, or been a member of any self-help  group for alcohol abuse?",
				"position": "newLine",
				"dependence": {
					"alcohol": "yes"
				},
				"defaultValue": "no",
				"values": [
					{
						"value": "no",
						"text": "No"
					},
					{
						"value": "yes",
						"text": "Yes"
					},
					{
						"value": "unknown",
						"text": "Unknown"
					}
				],
				"valueComparison": [{
					"value": "yes",
					"action": "submit"
				},{"value": "unknown",
				   "action": "submit"}]
			},
			{
				"type": "select",
				"ref": "chestPainDiagnosed",
				"label": "Was your chest pain medically diagnosed as heartburn, indigestion, or a muscle strain?",
				"position": "newLine",
				"dependence": {
					"chestPain": "yes"
				},
				"defaultValue": "no",
				"values": [
					{
						"value": "no",
						"text": "No"
					},
					{
						"value": "yes",
						"text": "Yes"
					},
					{
						"value": "unknown",
						"text": "Unknown"
					}
				],
				"valueComparison": {
					"value": "yes",
					"action": "occupation"
				}
			}

where here we simply want the answer of "yes" to this question to skip all intervening questions and head straight to the "occupation" questioning.

**  One observation is that some of the testing sequences that look like this:

			{
				"type": "select",
				"ref": "lastSeizure",
				"label": "When was your last seizure?",
				"defaultValue": "",
				"dependence": {
					"seizure": "yes"
				},
				"values": [
					{
						"value": "0",
						"text": "0 - 5 years"
					},
					{
						"value": "1",
						"text": "6 - 10 years"
					},
					{
						"value": "2",
						"text": "11 - 15 years"
					},
					{
						"value": "3",
						"text": "16 - 20 years"
					}
				],
				"valueComparison": [
				   {
					"value": "1",
					"action": "occupation"
				   },
				   {
					"value": "2",
					"action": "occupation"
				   },
				   {
					"value": "3",
					"action": "occupation"
				   }
				]
			},
			{
				"type": "select",
				"ref": "howManySeizures",
				"label": "How many seizures have you had in the past 12 months?",
				"defaultValue": "",
				"dependence": {
					"seizure": "yes"
				},
				"values": [
					{
						"value": "0",
						"text": "0 - 2"
					},
					{
						"value": "1",
						"text": "3 - 4"
					},
					{
						"value": "2",
						"text": "5 - 6"
					},
					{
						"value": "3",
						"text": "7+"
					}
				],
				"valueComparison": [
				   {
					"value": "3",
					"action": "submit"
				   }
				]
			}

could have been also done by providing some form of range checking in the valueComparison field as opposed to only supporting discrete values but there didn't seem to be enough payback for making a special case out of it. So there are a few that have longer lists, like this:

			{
				"type": "select",
				"ref": "lastSeizure",
				"label": "When was your last seizure?",
				"defaultValue": "",
				"dependence": {
					"seizure": "yes"
				},
				"values": [
					{
						"value": "0",
						"text": "0 - 5 years"
					},
					{
						"value": "1",
						"text": "6 - 10 years"
					},
					{
						"value": "2",
						"text": "11 - 15 years"
					},
					{
						"value": "3",
						"text": "16 - 20 years"
					}
				],
				"valueComparison": [
				   {
					"value": "1",
					"action": "occupation"
				   },
				   {
					"value": "2",
					"action": "occupation"
				   },
				   {
					"value": "3",
					"action": "occupation"
				   }
				]
			}

but the length of the lists was never bad enough to force more expressiveness.


In the reflexive sequences, there are cases where there is no response that causes a "next" question to asked:

			{
				"type": "select",
				"ref": "prescribedDepression",
				"label": "In the past 2 years, have you been prescribed medication by a medical professional for this condition?",
				"position": "newLine",
				"dependence": {
					"depressionMentalStress": "yes"
				},
				"defaultValue": "no",
				"values": [
					{
						"value": "no",
						"text": "No"
					},
					{
						"value": "yes",
						"text": "Yes and still taking this"
					},
					{
						"value": "0",
						"text": "Yes but stopped all medication with medical professional's advice"
					},
					{
						"value": "1",
						"text": "Yes but stopped taking medication on my own"
					}
				],
				"valueComparison": [
				  {
					"value": "1",
					"action": "submit"
				  },
				  {
					"value": "unknown",
					"action": "submit"
				  },
				  {
					"value": "no",
					"action": "occupation"
				  },
				  {
					"value": "0",
					"action": "occupation"
				  }
				]
			}

As can be seen above, all valid response cause some deviation from moving to the next question (typical of the end of a conditional reflexive).

** Note too that there is really no difference from a "select" question type and a "input" with "radiobox" subtype.

** There is a unique complexity in the 10T sequence where we need to allow the user to select "all that apply" from a set of choices. In order to support that, we have:

			{
				"type": "checkboxCollection",
				"label": "In the past 10 years, have you been medically diagnosed or medically treated for (select all that apply):",
				"position": "newLine",
				"refCollection": ["depressionMentalStress",
								"alcohol",
								"chestPain",
								"diabetes",
								"strokeTia",
								"seizure"],
				"valueComparison": {
					"moreCheckedThan": "1",
					"action": "submit"
				}
			}

the concept of a "checkboxCollection" structure which provides the general header for the question and a list ("refCollection") of the specific questions that make up the total set of choices. It also indicates in the "valueComparison" field that if more than one of the questions is checked to submit back to the server.

Then for each of those specific questions, there is a structure to define it:

			{
				"type": "input",
				"subType": "radiobox",
				"ref": "depressionMentalStress",
				"label": "(1) depression / mental disorder, ",
				"position": "newLine"
			},
			{
				"type": "input",
				"subType": "radiobox",
				"ref": "alcohol",
				"label": "(2) drug or alcohol abuse, ",
				"position": "newLine"
			},
			{
				"type": "input",
				"subType": "radiobox",
				"ref": "chestPain",
				"label": "(3) chest pains,",
				"position": "newLine"
			},
			{
				"type": "input",
				"subType": "radiobox",
				"ref": "diabetes",
				"label": "(4) diabetes,",
				"position": "newLine"
			},
			{
				"type": "input",
				"subType": "radiobox",
				"ref": "strokeTia",
				"label": "(5) stroke / TIA, or",
				"position": "newLine"
			},
			{
				"type": "input",
				"subType": "radiobox",
				"ref": "seizure",
				"label": "(6) seizure disorder",
				"position": "newLine"
			}

Note that the "ref" field of each is using one of the names from the "refCollection" list. These "ref" field definitions gives us the place to hold the response in the answer data, but is also used to control the opening of the reflexive sequence that we go through for the one that has been selected (remembering that only one of them can be selected if we are going to go through the reflexive sequence).

Thus, the first question in the reflexive sequence related to alcohol is declared with:

			{
				"type": "select",
				"ref": "alcoholAbuseTreatment",
				"label": "in the past 10 years, have you been treated or advised to get treatment by a medical professional, or been a member of any self-help  group for alcohol abuse?",
				"position": "newLine",
				"dependence": {
					"alcohol": "yes"
				},
				"defaultValue": "no",
				"values": [
					{
						"value": "no",
						"text": "No"
					},
					{
						"value": "yes",
						"text": "Yes"
					},
					{
						"value": "unknown",
						"text": "Unknown"
					}
				],
				"valueComparison": [{
					"value": "yes",
					"action": "submit"
				},{"value": "unknown",
				   "action": "submit"}]
			}

Noting that the "dependence" field is refering to the same "ref" field discussed about when we built the "refCollection".

** There were a few places where it was necessary for the frontend to evaluate data in the answer data as in:

			{
				"type": "answerTest",
				"ref": "diabetesGender",
				"refToTest": "gender",
				"valueComparison": 
				[
				  {
					"value": "male",
					"action": "diabetesMale"
				  },
				  {"value": "female",
				   "action":"diabetesFemale"
				  }
				]
			}

where here we are declaring a structure of type "answerTest". This structure has its own "ref" field so that it can be referenced elsewhere but it also has a "refToTest" field. In this case it is directing a comparison of the "gender" answer causing the next question to be either one with a "ref" field of diabetesMale or diabetesFemale as appropriate.

A variation on this idea where we need to compare to a numeric value against a previous answer. Similar to the earlier example, here we are comparing to an "age" field such that we submit back to the server if the user is less than 40 years old.

			{
				"type": "answerTestNumeric",
				"ref": "diabetesMale",
				"refToTest": "age",
				"valueComparison": {
					"value": "<40",
					"action": "submit"
				}
			}

It should be noted in this case that there really isn't a question that would have captured the age (rather we capture the birthdate on the previous screen). But it is my intention to have the placed in the answer data be the backend when he is processing the birthdate so that it will be available to the frontend on this next "page".


