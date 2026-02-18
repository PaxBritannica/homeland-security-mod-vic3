# # ADD HERE ANY FUTURE HOSTILE ACTIONS THAT COULD BE REVEALED
CHECK COUNTERINTELLIGENCE SWEEP EVENT 20 AND REMEMBER TO ADD ANY NEW HOSTILE ACTION THERE IN ORDER
FOR IT TO BE DETECTABLE BY THE SWEEP







# If a journal entry exist add progress to its scripted bars
if = {
    limit = {
        has_journal_entry = hms_je_sabotage
    }
    je:hms_je_sabotage ?= {
        add_progress = { value = 2 name = sabotage_police_progress_bar }
    }
}

# Pick a random country that exist and has relationship higher than cold with our country 
# make sure it is not root or decentralized
# is strategically adjacent or has interest in us
random_country = {
    limit = {

        NOT = { THIS = ROOT }
        NOT = { is_country_type = decentralized }
        has_strategic_adjacency = root
        relations:root >= relations_threshold:cold

    }
    save_scope_as = helper_country_1
}

# Pick the other target of a specific pact, MAKE SURE PACT is BILATERAL in is_two_sided_pact = yes
# Here prev can be replace with root 
 if = {
    limit = {
        any_country = {
            root = {
                has_diplomatic_pact = {
                    who = prev
                    type = encourage_piracy
                }
            }
        }
    }
    random_country = {
        limit = {
            root = {
                has_diplomatic_pact = {
                    who = prev
                    type = encourage_piracy
                }
            }
        }
        save_scope_as = piracy_initiator
    }
}
else = {
    random_country = {
        limit = {
            root = {
                has_diplomatic_pact = {
                    who = prev
                    type = encourage_piracy
                }
            }
        }
        save_scope_as = piracy_initiator
    }
}

root = {
    save_scope_as = piracy_target
}

# Change relationship with a scoped country
change_relations = {
    country = scope:helper_country_1
    value = 25
}

# Localization variables
[SCOPE.gsInterestGroup('reckless_out_of_govt_ig').GetNameNoFormatting] # Get an IG through var
[SCOPE.sCharacter('reckless_out_of_govt_leader').GetFullName] # Get a character from a set var


# compare variable
if = {
    limit = {
        var:armed_radical_pressure >= 3
    }
    
}

# Set global variable and retrieve it 
# usefult to cross reference countries and other var across journals or events
set_global_variable = {
    name = some_global_var
    target = this
}

global_var:some_global_var = {
    # do something in this country scope
}
        

#
# Useful to check if a SPECIFIC country has a SPECIFIC PACT with ANY COUNTRY
NOR = {

    #IF TARGET COUNTRY HAS ALREADY AN ECONOMIC ADVISORS IN ACT
    scope:target_country = {
        any_scope_diplomatic_pact = {
            is_diplomatic_action_type = send_economic_advisors
        }
    }

    root = {
        any_scope_diplomatic_pact = {
            is_diplomatic_action_type = send_economic_advisors
        }
    }
    
}

# maxe X ig unappeased 

ig:ig_industrialists ?= {
	    	join_revolution = yes
	        add_modifier = {
	            name = ig_unappeased
	            months = 36
	        }
		}


# Send globals across unrelated events using global vars and removing them asap so they do not suffer overlapping
# Setup quick global variable and immediately removed them once the event gets fired from the bloc leader
# Remember to do it before actually triggering the event

		set_global_variable = {
			name = country_ban_target_global
			value = scope:send_economic_advisors_target
		}
		set_global_variable = {
			name = country_investor_global
			value = this
		}

# call them back in the event immediate = {} block
immediate = {

		# grab global vars ()
		every_country = {
			limit = { this = global_var:country_investor_global }
			save_scope_as = ban_funder
		}

		every_country = {
			limit = { this = global_var:country_ban_target_global }
			save_scope_as = ban_target
		}

		# remove them asap ()
		remove_global_variable = country_investor_global
		remove_global_variable = country_ban_target_global

	}


# Hit approval of random government IG
        random_interest_group = {
            limit = {
                is_in_government = yes
            }
            add_modifier = {
                name = hms_ig_distrusts_intelligence
                months = 12
            }
        }

## Notification system for major diplo actions
 military_assistance_action_notification_third_party_name: "[concept_military_assistance] to [TARGET_COUNTRY.GetName]"
 military_assistance_action_notification_third_party_desc: "[INITIATOR_COUNTRY.GetName] is providing [concept_military_assistance] to [TARGET_COUNTRY.GetName]"


### save the root inside anothers country scope 
root scope 
    scope:target_country = {
        set_variable = {
            name = caught_by
            value = prev # or root
    }
}
# we use this to trigger the reaction event in the country that got caught 
scope:target_country = {
        set_variable = {
        name = caught_by
        value = prev
    }
    trigger_event = { id = reaction_generic_event.1 }
}

# then in immediate of the reaction event we do
immediate = {
        root.var:caught_by = {
            save_scope_as = caught_by
        }
    }