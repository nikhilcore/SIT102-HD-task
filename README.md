# SIT102-HD-task
In this repository i have shared my HD task for SIT102, in which i have made a game in which we have given some cases and we have to solve them using our mind and the objective of the game id to find the culprit.
#include "splashkit.h"
#include <string>
#include <vector>

using namespace std;

struct Suspect
{
    int id;
    string name;
    string description;
};

struct Evidence
{
    int id;
    string title;
    string description;
};

struct Statement
{
    int id;
    int suspectId;
    string text;
};

struct Contradiction
{
    int id;
    int statementId;
    int evidenceId;
    string description;
};

struct CaseFile
{
    int id;
    string title;
    string introText;
    int correctCulpritId;

    vector<Suspect> suspects;
    vector<Evidence> evidenceList;
    vector<Statement> statements;
    vector<Contradiction> contradictions;
};

struct CaseProgress
{
    int caseId;
    vector<int> unlockedStatements;
    vector<int> unlockedEvidence;
    vector<int> foundContradictions;
    int mistakes = 0;
    bool solved = false;
    bool accused = false;
    int finalAccusedId = -1;
};

void wait_for_enter()
{
    write_line("");
    write("Press ENTER to continue...");
    read_line();
}

int read_int_in_range(int min_val, int max_val)
{
    while (true)
    {
        string line = read_line();
        if (line.empty())
        {
            write("Enter a number between " + to_string(min_val) + " and " + to_string(max_val) + ": ");
            continue;
        }

        bool ok = true;
        int start = 0;
        if (line[0] == '+' || line[0] == '-')
        {
            if (line.length() == 1) ok = false;
            else start = 1;
        }

        for (int i = start; i < (int)line.length(); ++i)
        {
            if (!isdigit(line[i]))
            {
                ok = false;
                break;
            }
        }

        if (!ok)
        {
            write("Invalid input. Enter a number between " + to_string(min_val) + " and " + to_string(max_val) + ": ");
            continue;
        }

        int value = convert_to_integer(line);
        if (value < min_val || value > max_val)
        {
            write("Invalid input. Enter a number between " + to_string(min_val) + " and " + to_string(max_val) + ": ");
            continue;
        }

        return value;
    }
}

bool contains(const vector<int> &v, int value)
{
    for (int x : v)
        if (x == value) return true;
    return false;
}

int find_suspect_index(const CaseFile &c, int suspectId)
{
    for (int i = 0; i < (int)c.suspects.size(); i++)
    {
        if (c.suspects[i].id == suspectId) return i;
    }
    return -1;
}

int find_statement_index(const CaseFile &c, int statementId)
{
    for (int i = 0; i < (int)c.statements.size(); i++)
    {
        if (c.statements[i].id == statementId) return i;
    }
    return -1;
}

int find_evidence_index(const CaseFile &c, int evidenceId)
{
    for (int i = 0; i < (int)c.evidenceList.size(); i++)
    {
        if (c.evidenceList[i].id == evidenceId) return i;
    }
    return -1;
}

int find_contradiction_index(const CaseFile &c, int statementId, int evidenceId)
{
    for (int i = 0; i < (int)c.contradictions.size(); i++)
    {
        if (c.contradictions[i].statementId == statementId &&
            c.contradictions[i].evidenceId == evidenceId)
        {
            return i;
        }
    }
    return -1;
}

void init_case_progress(const CaseFile &c, CaseProgress &progress)
{
    progress.caseId = c.id;
    progress.unlockedStatements.clear();
    progress.unlockedEvidence.clear();
    progress.foundContradictions.clear();
    progress.mistakes = 0;
    progress.solved = false;
    progress.accused = false;
    progress.finalAccusedId = -1;

    for (int i = 0; i < (int)c.statements.size(); i++)
    {
        progress.unlockedStatements.push_back(c.statements[i].id);
    }

    for (int i = 0; i < (int)c.evidenceList.size(); i++)
    {
        progress.unlockedEvidence.push_back(c.evidenceList[i].id);
    }
}

void view_suspects(const CaseFile &c)
{
    write_line("");
    write_line("Suspects");
    for (int i = 0; i < (int)c.suspects.size(); i++)
    {
        const Suspect &s = c.suspects[i];
        write_line("ID: " + to_string(s.id) + " | " + s.name);
        write_line("   " + s.description);
    }
}

void view_evidence(const CaseFile &c, const CaseProgress &progress)
{
    write_line("");
    write_line("Evidence");
    for (int i = 0; i < (int)c.evidenceList.size(); i++)
    {
        const Evidence &e = c.evidenceList[i];
        if (!contains(progress.unlockedEvidence, e.id)) continue;
        write_line("ID: " + to_string(e.id) + " | " + e.title);
        write_line("   " + e.description);
    }
}

void view_statements(const CaseFile &c, const CaseProgress &progress)
{
    write_line("");
    write_line("Statements");
    for (int i = 0; i < (int)c.statements.size(); i++)
    {
        const Statement &st = c.statements[i];
        if (!contains(progress.unlockedStatements, st.id)) continue;

        int sIndex = find_suspect_index(c, st.suspectId);
        string line = "Statement ID: " + to_string(st.id);
        if (sIndex != -1)
        {
            line += " | Suspect: " + c.suspects[sIndex].name;
        }
        write_line(line);
        write_line("   " + st.text);
    }
}

void present_evidence(const CaseFile &c, CaseProgress &progress)
{
    write_line("");
    write_line("Present Evidence");
    write("Choose a Statement ID (0 to cancel): ");
    int stId = read_int_in_range(0, 1000);
    if (stId == 0) return;

    if (!contains(progress.unlockedStatements, stId))
    {
        write_line("No such unlocked statement.");
        return;
    }

    int stIndex = find_statement_index(c, stId);
    if (stIndex == -1)
    {
        write_line("Statement not found.");
        return;
    }

    const Statement &st = c.statements[stIndex];

    write_line("");
    write_line("You selected:");

    int sIndex = find_suspect_index(c, st.suspectId);
    string header = "Statement ID " + to_string(st.id);
    if (sIndex != -1)
    {
        header += " by " + c.suspects[sIndex].name;
    }
    write_line(header);
    write_line("  \"" + st.text + "\"");

    write_line("");
    write("Now choose Evidence ID to present (0 to cancel): ");
    int evId = read_int_in_range(0, 1000);
    if (evId == 0) return;

    if (!contains(progress.unlockedEvidence, evId))
    {
        write_line("No such unlocked evidence.");
        return;
    }

    int evIndex = find_evidence_index(c, evId);
    if (evIndex == -1)
    {
        write_line("Evidence not found.");
        return;
    }

    const Evidence &ev = c.evidenceList[evIndex];

    write_line("");
    write_line("You present evidence \"" + ev.title + "\" against that statement...");

    int conIndex = find_contradiction_index(c, stId, evId);
    if (conIndex == -1)
    {
        write_line("That evidence does not contradict the statement.");
        progress.mistakes++;
        write_line("Mistakes so far: " + to_string(progress.mistakes));
        return;
    }

    int conId = c.contradictions[conIndex].id;
    if (contains(progress.foundContradictions, conId))
    {
        write_line("You already found this contradiction before.");
        return;
    }

    progress.foundContradictions.push_back(conId);

    const Contradiction &con = c.contradictions[conIndex];

    write_line("");
    write_line("CONTRADICTION FOUND");
    write_line(con.description);
    write_line("Number of contradictions found: " + to_string(progress.foundContradictions.size()));
}

void accuse_suspect(const CaseFile &c, CaseProgress &progress)
{
    if (progress.accused)
    {
        write_line("You already made an accusation in this case.");
        return;
    }

    write_line("");
    write_line("Accuse a Suspect");
    view_suspects(c);
    write("Enter Suspect ID to accuse (0 to cancel): ");
    int sId = read_int_in_range(0, 1000);
    if (sId == 0) return;

    int sIndex = find_suspect_index(c, sId);
    if (sIndex == -1)
    {
        write_line("No such suspect.");
        return;
    }

    progress.accused = true;
    progress.finalAccusedId = sId;

    const Suspect &sus = c.suspects[sIndex];

    write_line("");
    write_line("You accuse " + sus.name + "...");

    if (sId == c.correctCulpritId)
    {
        write_line("They are the true culprit.");
        progress.solved = true;
    }
    else
    {
        write_line("That is the wrong culprit.");
        progress.solved = false;
    }
}

void show_case_summary(const CaseFile &c, const CaseProgress &progress)
{
    write_line("");
    write_line("CASE SUMMARY");
    write_line("Case: " + c.title);

    if (!progress.accused)
    {
        write_line("You never made a final accusation.");
    }
    else
    {
        int accIndex = find_suspect_index(c, progress.finalAccusedId);
        int culIndex = find_suspect_index(c, c.correctCulpritId);

        string acc_name = (accIndex != -1) ? c.suspects[accIndex].name : string("(invalid suspect)");
        string cul_name = (culIndex != -1) ? c.suspects[culIndex].name : string("(unknown)");

        write_line("Your accusation: " + acc_name);
        write_line("True culprit: " + cul_name);
    }

    write_line("Contradictions found: " +
               to_string(progress.foundContradictions.size()) +
               " out of " + to_string(c.contradictions.size()));
    write_line("Mistakes made: " + to_string(progress.mistakes));

    string rating = "D";
    int totalCon = (int)c.contradictions.size();
    int foundCon = (int)progress.foundContradictions.size();

    if (progress.solved && foundCon == totalCon && progress.mistakes == 0)
        rating = "S";
    else if (progress.solved && foundCon >= totalCon / 2)
        rating = "A";
    else if (progress.solved)
        rating = "B";
    else if (!progress.solved && foundCon >= totalCon / 2)
        rating = "C";

    write_line("Detective Rank: " + rating);
}

CaseFile create_case_1()
{
    CaseFile c;
    c.id = 1;
    c.title = "The Broken Lab Window";
    c.introText =
        "A lab window was smashed last night, and an expensive device is missing.\n"
        "Only three people were in the building at the time.\n";
    c.correctCulpritId = 2;

    c.suspects = {
        {1, "Aisha", "Lab assistant. Claims she left early."},
        {2, "Rohan", "Research student. Often stays late."},
        {3, "Meera", "Security guard. Responsible for locking doors."}
    };

    c.evidenceList = {
        {1, "Access Log",
         "Door access card records show Rohan entered at 21:10 and left at 23:55."},
        {2, "Camera Snapshot",
         "CCTV still at 23:30 shows someone in a hoodie at the lab door."},
        {3, "Glass Shards on Inside",
         "Most broken glass is found inside the lab, not outside."},
        {4, "Meera's Logbook",
         "Security log shows 'All clear, doors locked' at 23:00."}
    };

    c.statements = {
        {1, 1, "I left the building by 22:00. I didn't hear any glass breaking."},
        {2, 2, "Yes, I was in the lab, but I left before 23:00. The window was fine."},
        {3, 3, "I locked all doors at exactly 23:00. No one was inside after that."},
        {4, 2, "If the window broke, it must have been from outside. Maybe a random vandal."}
    };

    c.contradictions = {
        {1, 2, 1, "Access log shows Rohan left at 23:55, not before 23:00 as he claimed."},
        {2, 3, 1, "Security cannot claim no one was inside after 23:00 because Rohan left at 23:55."},
        {3, 4, 3, "Glass inside the lab suggests the window was broken from inside, not by an outsider."}
    };

    return c;
}

CaseFile create_case_2()
{
    CaseFile c;
    c.id = 2;
    c.title = "Missing Exam Paper";
    c.introText =
        "The final exam paper for Algorithms has disappeared from the department office.\n"
        "Three people had access to the room.\n";
    c.correctCulpritId = 1;

    c.suspects = {
        {1, "Karan", "Topper of the class, obsessed with perfect grades."},
        {2, "Prof. Sharma", "Course instructor, wrote the exam."},
        {3, "Neha", "Department admin, handles printing and copying."}
    };

    c.evidenceList = {
        {1, "Printer Queue Log",
         "Log shows a large print job from Neha's account at 18:45."},
        {2, "CCTV Corridor",
         "Camera shows Karan entering the office at 19:10, leaving at 19:13."},
        {3, "Locked Drawer",
         "Drawer where exam papers were kept shows no signs of forced entry."},
        {4, "Spare Key Record",
         "Record shows one spare key missing from key box, signed out by 'K' with no full name."}
    };

    c.statements = {
        {1, 1, "I never went near the office yesterday evening. I was in the library."},
        {2, 2, "I locked the exam papers in the drawer and left at 18:30."},
        {3, 3, "I printed some material at 18:45, but I didn't touch the exam papers."},
        {4, 1, "Even if I wanted the paper, I don't have any key to that drawer."}
    };

    c.contradictions = {
        {1, 1, 2, "CCTV shows Karan entering the office at 19:10, contradicting his claim."},
        {2, 2, 1, "If Sharma left at 18:30, printer log at 18:45 shows Neha was still around, not him."},
        {3, 4, 4, "Spare key signed out by 'K' suggests Karan could access the locked drawer."},
        {4, 3, 1, "Neha admits printing, but nothing proves she accessed the locked drawer."}
    };

    return c;
}

void play_case(CaseFile &c)
{
    CaseProgress progress;
    init_case_progress(c, progress);

    write_line("");
    write_line("CASE " + to_string(c.id) + ": " + c.title);
    write_line(c.introText);

    bool running = true;
    while (running)
    {
        write_line("");
        write_line("Case Menu");
        write_line("1. View suspects");
        write_line("2. View evidence");
        write_line("3. View statements");
        write_line("4. Present evidence");
        write_line("5. Accuse a suspect");
        write_line("6. Show case summary");
        write_line("0. Exit this case");
        write("Choose an option: ");

        int choice = read_int_in_range(0, 6);

        if (choice == 1)
            view_suspects(c);
        else if (choice == 2)
            view_evidence(c, progress);
        else if (choice == 3)
            view_statements(c, progress);
        else if (choice == 4)
            present_evidence(c, progress);
        else if (choice == 5)
            accuse_suspect(c, progress);
        else if (choice == 6)
            show_case_summary(c, progress);
        else if (choice == 0)
            running = false;
    }

    write_line("");
    write_line("Final summary for this case:");
    show_case_summary(c, progress);
    wait_for_enter();
}

void main_menu(vector<CaseFile> &cases)
{
    bool running = true;
    while (running)
    {
        write_line("");
        write_line("LOGIC DETECTIVE");
        write_line("Available cases:");
        for (int i = 0; i < (int)cases.size(); i++)
        {
            write_line("  " + to_string(cases[i].id) + ". " + cases[i].title);
        }
        write_line("  0. Exit game");
        write("Choose a case ID to investigate: ");

        int choice = read_int_in_range(0, 1000);
        if (choice == 0)
        {
            running = false;
        }
        else
        {
            bool found = false;
            for (int i = 0; i < (int)cases.size(); i++)
            {
                if (cases[i].id == choice)
                {
                    play_case(cases[i]);
                    found = true;
                    break;
                }
            }
            if (!found)
            {
                write_line("No such case.");
            }
        }
    }
}

int main()
{
    vector<CaseFile> cases;
    cases.push_back(create_case_1());
    cases.push_back(create_case_2());

    write_line("Game starting...");
    main_menu(cases);
    write_line("");
    write_line("Thank you for playing Logic Detective.");
    return 0;
}
