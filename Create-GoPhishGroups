<#
This script will create GoPhish groups based off of membership in an AD group.
It will create the group with the same name as the AD group.
Written by:
Shane Fonyi
The University of Kansas
IT Security Office
10/25/2017
#>
#Requires –Modules ActiveDirectory
# We first need to specify which AD groups we want to work with separated by commas
$Groups="AdjunctFaculty","AffiliatedCorporation","Classified","Emeritus","Faculty","GraduateAssistant","GraduateResearchAssistant","GraduateTeachingAssistant","IncomingStaff","Retiree","StudentStaff","UnclassifiedAcademicStaff","UnclassifiedProfessionalStaff"
#Initizalize the group counter
$count= 1
#Get date in Round Trip format
$time = get-date -format o
#Set the URL for the GoPhish instance with the default port of 3333
#example: gophish.domain.com
$url=
#set the api key
$api=

foreach ($group in $Groups){
write-host $group
# Next we need to initialize our object counter
$counter = 1

# Next we need to pull users from AD with their first name, last name, email, and department (title can be used instead)
$output = Get-ADGroupMember -identity $group | `
        Get-ADUser -Properties mail, Title, Department, GivenName, Surname | `
        Select-Object ('mail', 'GivenName', 'Surname', 'Title') | `
        Select `
        @{Name="first_name";Expression={$_."GivenName"}},
        @{Name="last_name";Expression={$_."Surname"}},
        @{Name="email";Expression={$_."mail"}},
        @{Name="position";Expression={$_."Title"}}


# Next we need to put the output into a format that GoPhish can read. We use an array in an array to do this. The top array has the "headers" for GoPhish.
# The next array inside the targets has the users with data.
$body = @{
    id = $count
    name = "$group"
    modified_date = "$time"
    targets = $output | Select-Object @{ Name = "id" ; Expression= {$global:counter; $global:counter++} }, first_name, last_name, email, position

    }

# Lastly we need to convert it to ascii so that gophish can read it and send it via api.
$body = $body | ConvertTo-Json
$body | Out-File  -Encoding ascii "$PSScriptRoot\targets.json"
$filecontent = Get-Content '$PSScriptRoot\targets.json'
write-host $count
#Now we will delete the existing group
Invoke-RestMethod -Uri "http://$url:3333/api/groups/$count`?api_key=$api" -Method Delete
#Finally we will create our new updated group
Invoke-RestMethod -Uri "http://$url:3333/api/groups/?api_key=$api" -ContentType "application/json" -Method POST -body ([System.Text.Encoding]::ASCII.GetBytes($filecontent))
$count++
}
