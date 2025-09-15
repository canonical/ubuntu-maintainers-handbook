# Distro Skill Tree

This page contains an interactive chart for navigating paths to obtaining
permissions to upload changes to Ubuntu archive. This can be used as a model
to help build an application for things such as certain packagesets, 
[MOTU](https://wiki.ubuntu.com/MOTU), 
[CoreDev](https://wiki.ubuntu.com/UbuntuDevelopers#Ubuntu_Core_Developers) 
as examples.

```mermaid
%% mermaid flowcharts documentation: https://mermaid.js.org/syntax/flowchart.html
%%{ init: { 'flowchart': { 'curve': 'catmullRom' } } }%%
flowchart TD

    Start((" ")):::Invisible
    Start --> |"Path to Distro Contribution"| Basics

    subgraph Basics
        direction TB
        subgraph InitialStudies["Initial Studies"]
            direction BT
            %% Concepts{{"Concepts"}}
            Concepts{{"<a href=https://github.com/canonical/ubuntu-maintainers-handbook>Concepts</a>"}}:::study
            Git-Ubuntu{{"Git-Ubuntu"}}:::study
            Debian-Policy{{"<a href=https://www.debian.org/doc/debian-policy/>Debian Policy</a>"}}:::study
        end
        subgraph InitialTasks["Initial Tasks"]
            direction BT
            BiteSizedBugs((Bite Sized Bugs)):::task
            TrivialPackgeMerges(("Trivial Package Merges")):::task
        end
    end

    InitialStudies --> InitialTasks

    BasicsToIntermediate{"Team/Mentor Says ready for more"}:::concept

    Basics --> BasicsToIntermediate --> Intermediate
    subgraph Intermediate
        direction TB
        subgraph IntermediateTasks[Intermediate Tasks]
            direction TB
            %% States
            ComplexPackageMerges(("Complex Package Merges")):::task
            ProposeMigration(("<a href=https://wiki.ubuntu.com/ProposedMigration>Proposed Migration</a>")):::task
            UnderstandDep8{{"<a href=https://salsa.debian.org/ci-team/autopkgtest/blob/master/doc/README.package-tests.rst>Understand DEP8</a>"}}:::study
            AddAUTOPKGTESTS(("<a href=https://github.com/canonical/ubuntu-maintainers-handbook/blob/main/PackageTests.md>Add Autopkgtest</a>")):::task
            SRU{{"<a href=https://canonical-sru-docs.readthedocs-hosted.com/>Study SRU</a>"}}:::study
            DoSRUS(("Do SRUS")):::task

            %% Transitions
            UnderstandDep8 --> AddAUTOPKGTESTS
            ComplexPackageMerges --> ProposeMigration
            SRU --> DoSRUS
        end
        IntermediateKeepGoing["Do enough of these to apply for package or group uploads"]:::task
        IntermediateTasks --> IntermediateKeepGoing --> IntermediateTasks
    end

    IntermediateToAdvanced{"Team/Mentor Says ready for more"}:::concept
    Intermediate --> IntermediateToAdvanced --> Advanced

    subgraph Advanced
    direction LR
        subgraph AdvancedTasks[Advanced Tasks]
            direction LR
            %% States
            UpstreamSubmissionFixes(("Upstream Submission Fixes/Features")):::task
            UpstreamSubmissionDelta(("Upstream Submission of Delta")):::task
            MilestonesAndExceptions(("Milestones And Exceptions")):::task
            StudyFFE{{"<a href=https://wiki.ubuntu.com/FreezeExceptionProcess>Study FFE</a>"}}:::study
            DoAnFFE(("Do An FFE")):::task
            PlusOne{{"<a href=https://wiki.ubuntu.com/PlusOneMaintenanceTeam>Study +1</a>"}}:::study
            PlusOneShadowing(("+1 Shadowing")):::task

            %% Transitions
            StudyFFE-->DoAnFFE
            PlusOne-->PlusOneShadowing
        end
        AdvancedKeepGoing["Do enough of these to apply for MOTU"]:::task
        AdvancedTasks --> AdvancedKeepGoing --> AdvancedTasks
    end

    Advanced --> optionalDebian
    MOTU{"<a href=https://github.com/canonical/ubuntu-maintainers-handbook/blob/main/MembershipInMOTU.md>MOTU</a>"}:::concept
    Advanced --> MOTU --> Expert

    subgraph optionalDebian[Optional Activites in Debian]
        %% States
        Contribute(("<a href=https://www.debian.org/doc/manuals/maint-guide/>Contribute</a>")):::task
        DM{"<a href=https://wiki.debian.org/DebianMaintainer>DM</a>"}:::concept
        DD{"<a href=https://wiki.debian.org/DebianDeveloper>DD</a>"}:::concept

        %% Transitions
        Contribute --> DM
        DM --> DD
    end

    subgraph Expert
        direction LR
        subgraph ExpertTasks
            direction TB

            %% States
            StudyLibaryTransitions{{"<a href=https://wiki.debian.org/Teams/ReleaseTeam/Transitions>Study Libary Transitions</a>"}}:::study
            DoLibaryTransitions(("Do Libary Transitions")):::task
            StudyPackageTransitions{{"<a href=https://wiki.debian.org/PackageTransition>Study Package Transitions</a>"}}:::study
            DoPackageTransitions(("Do Package Transitions")):::task
            StudyMIR{{"<a href=https://github.com/canonical/ubuntu-mir/edit/main/README.md>Study MIR</a>"}}:::study
            DoMIR(("Do a MIR")):::task
            SeedChange(("Seed Change")):::task

            %% Transitions
            StudyLibaryTransitions-->DoLibaryTransitions
            StudyPackageTransitions-->DoPackageTransitions
            StudyMIR-->DoMIR
            StudyMIR-->SeedChange
        end
        ExpertKeepGoing["Do enough to apply for core-dev"]:::task
        ExpertTasks-->ExpertKeepGoing-->ExpertTasks
    end

    CoreDev{"<a href=https://github.com/canonical/ubuntu-maintainers-handbook/blob/main/MembershipInCoreDev.md>Core Developer</a>"}:::concept

    Expert --> CoreDev --> Duties

    subgraph Duties
        direction LR
        CoreDevPlusOne(("+1")):::task
        Sponsoring(("Sponsoring")):::task
        Mentoring(("Mentoring")):::task
    end
```
