# Einfacher-Vokabeltrainer
Hey, diesen Vokabeltrainer kannst du benutzen um Vokabeln in neuen Sprachen aufzuschreiben und abzufragen.
Er wurde komplett in QT 6.4 mit C++ geschrieben.
Dieses Projekt, ist mein kennlern-Projekt für QT.

Der Code inklusive Erklärung:

Disclaimer: Zur Veranschaulichung wurde der Code in nur ein einzelnes Dokument geschrieben, eigentlich ist der Code auf mehrere Dateien aufgeteilt. Hier wird auch nur der Code gezeigt, den ich selber geschrieben habe, der Code, der automatisch von QT generiert wird (HTML & der Großteil des CSS Codes), hingegen nicht.



⬇️ Hier werden C++ und QT Bibliotheken importiert.
#include <QPalette>
#include <QStyleFactory>
#include "mainwindow.h"
#include "./ui_mainwindow.h"
#include <iostream>
#include <string>
#include <QVector>
#include <fstream>
#include <iterator>
#include <cstdlib>
#include <ctime>
#include <sstream>
#include <QTextStream>
#include <QMouseEvent>
#include <QFile>
#include "functions.cpp"
#include <QRegularExpression>
#include <QDir>
#include <QTimer>
#include <QStatusBar>
#include <QIcon>

⬇️ Hier wird namespace std implementiert, was dafür sorgt, dass die Elemente im Standard-namespace ohne Namen Qualifizierung implementiert werden können. Mir ist bewusst, dass das gerade in großen Programmen nicht optimal ist, aber ich habe mich für dieses Projekt trotzdem dafür entschieden. 
using namespace std;



⬇️ Hier werden alle Globalen Variablen initialisiert. Später folgt zu jeder eine Erklärung.
QVector<QString> voc_my_lang;
QVector<QString> voc_lang_to_learn;
QVector<QString> last_languages = {"",""};
QString my_lang = last_languages[0];
QString lang_to_learn = last_languages[1];
qint16 which_file_to_edit = 0;
qreal voc_size_saved = 0;
QVector<QString> vocab_container_lang1 = {"",""};
QVector<QString> vocab_container_lang2 = {"",""};
bool first_time_user ;




⬇️ Mit dieser Funktion werden die Vektoren in die entsprechende Datenbank geschrieben. Heirbei sei angemerkt, dass ich txt Files als Datenbanken benutzt habe.
void write_text_file(const QString& textFile, const QVector<QString>& voc_container, bool append = false) {
    qInfo() << textFile+" wird geschrieben...";
    const QString dataDirPath = QCoreApplication::applicationDirPath() + "/data/";
    QDir dataDir(dataDirPath);

    if (!dataDir.exists() && !dataDir.mkpath(dataDirPath)) {
        qCritical() << "Kann Datenverzeichnis nicht erstellen: " << dataDirPath;
        return;
    }

    QFile file(dataDirPath + textFile);
    if (!file.open(append ? QIODevice::Append : QIODevice::WriteOnly | QIODevice::Text)) {
        qCritical() << "Kann Datei nicht öffnen: " << file.fileName();
        return;
    }

    QTextStream out(&file);

    for (const QString& str : voc_container) {
        out << str << '\n';
    }

    file.close();
}






⬇️ Mit dieser Funktion werden Daten aus der entsprechenden Datenbank geladen. Falls es keine Datei mit dem angegebenen Namen existiert, wird diese Datei dann erstellt.
QVector<QString> load_text_file(QString textFile){
    qInfo() << textFile+" wird geladen...";
    QString dataDirPath = QCoreApplication::applicationDirPath() + "/data/";
    QString filePath = dataDirPath + textFile;

    QFile file(filePath);
    QVector<QString> file_vector;
    if (!file.open(QIODevice::ReadOnly | QIODevice::Text)) {
        cerr << "Kann Datei nicht oeffnen: " << filePath.toStdString() << endl;
        write_text_file(textFile, file_vector);
        return load_text_file(textFile);
    }

    QTextStream in(&file);
    QString str;
    while (!in.atEnd())
    {
        str = in.readLine().trimmed();
        if(str.size() > 0){
            file_vector.append(str);
        }
    }

    file.close();
    return file_vector;
}



⬇️ Hier wird das graphische Interace gestartet
MainWindow::MainWindow(QWidget *parent)
    : QMainWindow(parent)
    , ui(new Ui::MainWindow)
{
    ui->setupUi(this);
    qInfo() << "Programm startet";

    setup_programm();

}

MainWindow::~MainWindow()
{
    delete ui;
}

⬇️ Hier werden weitere Teile des Programms gestartet.
void MainWindow::setup_programm(){
    enable_window_buttons();
    disable_tutorial_labels();
    setWindowTitle("Simons C++ Vokabeltrainer");
    ui->normalize_window->hide();
    setWindowFlags(Qt::FramelessWindowHint);
    last_languages = load_text_file("last_languages.txt");
    qInfo() << QString::number(last_languages.size());
    ui->header_languages->setVisible(false);

    define_languages();

    auto voc_containers = load_last_langs();
    voc_my_lang = voc_containers.first;
    voc_lang_to_learn = voc_containers.second;
}

⬇️ Falls der User nicht zum ersten mal das Programm benutzt, werden diese Labels unsichtbar.
void MainWindow::disable_tutorial_labels(){
    ui->menu_tutorial_label_settings->setVisible(false);
    ui->Settings_tutorial_label1->setVisible(false);
    ui->Settings_tutorial_label2->setVisible(false);
    ui->Settings_tutorial_label3->setVisible(false);
    ui->menu_tutorial_label_add_voc->setVisible(false);
    ui->tutorial_label_add_voc1->setVisible(false);
    ui->tutorial_label_add_voc2->setVisible(false);
    ui->menu_tutorial_label_voc_test->setVisible(false);
    ui->tutorial_label_voc_test1->setVisible(false);
    ui->tutorial_label_voc_test2->setVisible(false);

}

⬇️ Hier werden die Icons zum minimieren, maximieren und schließen der Anwendung importiert und gesetzt.
void MainWindow::enable_window_buttons(){
    QString iconsDirPath = QApplication::applicationDirPath() + "/icons/";

    QIcon closeIcon(iconsDirPath + "Close_Icon.svg");
    ui->close_window->setIcon(closeIcon);

    QIcon MaxIcon(iconsDirPath + "Max_Icon.svg");
    ui->max_window->setIcon(MaxIcon);

    QIcon minIcon(iconsDirPath + "Min_Icon.svg");
    ui->min_window->setIcon(minIcon);

    QIcon normalizeIcon(iconsDirPath + "Max_to_normal_Icon.svg");
    ui->normalize_window->setIcon(normalizeIcon);
}






⬇️ Hier wird ausgelesen, welche Sprachen die zuletzt benutzen waren.
pair<QVector<QString>, QVector<QString> > MainWindow::load_last_langs(){
    qInfo() << "die letzten Sprachen werden geladen...";
    QVector<QString> voc_my_lang;
    QVector<QString> voc_lang_to_learn;
        if(last_languages.size()<2){
            new_user(0);
        }else{
            my_lang = last_languages[0];
            lang_to_learn = last_languages[1];
            voc_my_lang = load_text_file(my_lang+"_"+lang_to_learn+".txt");
            voc_lang_to_learn = load_text_file(lang_to_learn+"_"+my_lang+".txt");
        }
    return make_pair(voc_my_lang, voc_lang_to_learn);
}




⬇️ Hier werden die letzten Sprachen global als my_lang und lang_to_learn definiert und dann ins Userface übernommen.
void MainWindow::define_languages(){
if(last_languages.size() > 1){
    qInfo() << "letzte Sprachen werden definiert";
    my_lang = last_languages[0];
    lang_to_learn = last_languages[1];
    setup_dropdown(true);
    set_languages_label();
    vocab_container_lang1 = load_text_file(my_lang+"_"+lang_to_learn+".txt");
    vocab_container_lang2 = load_text_file(lang_to_learn+"_"+my_lang+".txt");
    }
}



⬇️ Hier wird mit css festgelegt, wie ein Knopf aussehen sollte, sobald er gehighlighted wird.
QString MainWindow::get_highlighted_button_style(){
    QString style_highlighted_button =
            "QPushButton {"
                "color: rgb(230, 230, 255); "
                "background-color: rgb(30, 60, 100);"
                "border-style: solid;"
                "border-width: 2px;"
                "border-radius: 5px;"
                "border-color: rgb(230, 230, 255);"
                "}"

                "QPushButton:hover {"
                "background-color: rgb(50, 100, 150);"
                "border-style: solid;"
               " border-width: 3px;"
               " border-radius: 5px;"
                "border-color: rgb(230, 230, 255);"
                "}";
    return style_highlighted_button;
}




⬇️ Hier wird mit css festgelegt, wie ein Knopf aussehen sollte, wenn er nicht gehighlighted wurde.
QString MainWindow::get_normal_button_style(){
    QString style_normal_button =
            "QPushButton {"
                    "background-color: rgb(20, 50, 70); color: rgb(230, 230, 255); "
                     "}"

                      "QPushButton:hover {"
                      "background-color: rgb(30, 60, 100);"
                      "border-style: solid;"
                      "border-width: 2px;"
                      "border-radius: 5px;"
                      "border-color: rgb(230, 230, 255);"
                      "}";
    return style_normal_button;
}




⬇️ Hier wird die Einführung für neue Benutzer gestartet. DIese Funktion führt einen gewissen Teil der Einführung aus, je nachdem bei welchem Teil der Einführung der Benutzer aktuell ist. Es gibt Step 0-3, falls der input etwas außerhalb dieses Bereiches ist, wird die Einführung deaktiviert.
void MainWindow::new_user(qint16 step){
    QString style_normal_button = get_normal_button_style();
    QString style_highlighted_button = get_highlighted_button_style();

    if(step == 0){
        first_time_user = true;
        qInfo() <<"Neuer Benutzer";
        message_to_user("Hey, du scheinst neu zu sein. Gib am besten erst einmal unter Einstellungen deine Sprachen an! :D", true, 10000);
        ui->Menu_settings->setStyleSheet(style_highlighted_button);

        tutorial1(true);
    }else if(step == 1){
        ui->back_tomenu_button3->setStyleSheet(style_highlighted_button);
        ui->Settings_tutorial_label3->setVisible(true);


    }else if(step == 2){
        tutorial1(false);
        ui->back_tomenu_button3->setStyleSheet(style_normal_button);
        ui->Menu_settings->setStyleSheet(style_normal_button);
        ui->Menu_add_Vokabeln->setStyleSheet(style_highlighted_button);
        tutorial2(true);

    }else if(step == 3){
        tutorial2(false);
        ui->Menu_add_Vokabeln->setStyleSheet(style_normal_button);
        ui->Menu_Vokaltest_start->setStyleSheet(style_highlighted_button);
        tutorial3(true);

    }else{
        ui->Menu_Vokaltest_start->setStyleSheet(style_normal_button);
        disable_tutorial_labels();
        first_time_user = false;
    }
}

⬇️ Diese Funktion aktiviert / deaktiviert die Einführung Teil 1.
void MainWindow::tutorial1(bool enable){
    if(enable){
        ui->Menu_add_Vokabeln->setDisabled(true);
        ui->Menu_Vokaltest_start->setDisabled(true);
        ui->menu_tutorial_label_settings->setVisible(true);
        ui->Settings_tutorial_label1->setVisible(true);
        ui->Settings_tutorial_label2->setVisible(true);
    }else{
        ui->menu_tutorial_label_settings->setVisible(false);
        ui->Settings_tutorial_label1->setVisible(false);
        ui->Settings_tutorial_label2->setVisible(false);
        ui->Menu_add_Vokabeln->setDisabled(false);
        ui->Settings_tutorial_label3->setVisible(false);
    }
}



⬇️ Diese Funktion aktiviert / deaktiviert die Einführung Teil 2.
void MainWindow::tutorial2(bool enable){
    if(enable){
        ui->menu_tutorial_label_add_voc->setVisible(true);
        ui->tutorial_label_add_voc1->setVisible(true);
        ui->tutorial_label_add_voc2->setVisible(true);
        message_to_user("Hier, bei Vokabeln hinzufuegen, kannst du deine ersten Vokabeln hinzufuegen.", true, 15000);
    }else{
        ui->Menu_Vokaltest_start->setDisabled(false);
        ui->menu_tutorial_label_add_voc->setVisible(false);
        ui->tutorial_label_add_voc1->setVisible(false);
        ui->tutorial_label_add_voc2->setVisible(false);
    }
}




⬇️ Diese Funktion aktiviert die Einführung Teil 3. Teil 3 wird deaktiviert, wenn die ganze Einführung deaktiviert wird.
void MainWindow::tutorial3(bool enable){
    if(enable){
        ui->menu_tutorial_label_voc_test->setVisible(true);
        ui->tutorial_label_voc_test1->setVisible(true);
        ui->tutorial_label_voc_test2->setVisible(true);
        message_to_user("Du kannst nun versuchen einen Vokabeltest zu starten. Viel Glueck!", true, 10000);
    }
}



⬇️ Hier wird eine Nachricht an den Benutzer gesendet. Die EIngaben sind, die Nachricht, ob der Timer Neugestartet werden soll und wie lange die Nachricht da stehen soll in Millisekunden. 
void MainWindow::message_to_user(QString message, bool enabled,qint32 time_ms){

        statusBar()->setVisible(true);
        statusBar()->setSizeGripEnabled(false);
        statusBar()->setFixedHeight(30);

        static QTimer *m_timer = nullptr;

        QString style = "QStatusBar:hover {"
                        "background-color: rgb(20, 50, 70);"
                        "color: #38c6e0;"
                        "}"
                        "QStatusBar{"
                        "background-color: rgb(36, 64, 83);"
                        "color: #38c6e0;"
                        "font-family: 'Lucida Console';"
                        "font-size: 14px;"
                        "}"
                        "QStatusBar QLabel{"
                        "text-align: center"
                        "}";

        if(m_timer != nullptr && m_timer->isActive()) {
                m_timer->stop();
                m_timer->disconnect();
                m_timer->deleteLater();
        }
    if(enabled){
        statusBar()->setStyleSheet(style);
        statusBar()->showMessage(tr(message.toLatin1().constData()),time_ms);
        QTimer *timer = new QTimer(statusBar());
        connect(timer, &QTimer::timeout, statusBar(), &QStatusBar::hide);
        timer->start(time_ms);
    }else{
        statusBar()->setVisible(false);
    }
}





⬇️ Hier wird die Variable file mit den Benutzersprachen definiert, damit diese dann in den Dropdown in den Einstellungen geladen werden können.
QVector<QString> MainWindow::prepare_lang_txt_for_dropdown(qint16 lang_type, bool rmv){
    qInfo() << "die lang txt wird für den dropdown vorbereitet...";
    QString file_name = "User_Languages"+QString::number(lang_type)+".txt";
    QVector<QString> file = load_text_file(file_name);
    qint16 f_size = file.size();
    if(f_size> 0 && file[f_size-1] == "Neue Sprache"){
        if(rmv){
            file.removeLast();
        }
    }else if(!rmv){
        file.append("Neue Sprache");
    }
    return file;
}


⬇️ Hier werden die Dropdowns für die Sprachen in den Einstellungen gesetzt.
void MainWindow::setup_dropdown(bool hide_label){
    qInfo() << "die dropdowns werden vorbereitet...";
    QVector<QString> all_my_lang = prepare_lang_txt_for_dropdown(1, false);
    QVector<QString> all_lang_to_learn = prepare_lang_txt_for_dropdown(2, false);
        ui->lang2_dropdown->clear();
        ui->lang1_dropdown->clear();
        ui->lang2_dropdown->addItems(all_lang_to_learn);
        ui->lang2_dropdown->setCurrentText(lang_to_learn);
        ui->lang1_dropdown->clear();
        ui->lang1_dropdown->addItems(all_my_lang);
        ui->lang1_dropdown->setCurrentText(my_lang);
        ui->New_lang_input_field->setVisible(false);
    if(hide_label){
        ui->add_new_lang_label->setVisible(false);
    }
}




⬇️ Falls in den Einstellungen eine neue Sprache hinzugefügt wird, wird mit dieser Funktion die neue Sprache erstellt und automatisch als neue aktive Sprache ausgewählt.
void MainWindow::new_language(qint16 lang_type, QString new_lang){
    qInfo() << "neue Sprache wird hinzugefügt...";
    prepare_lang_txt_for_dropdown(lang_type, true);
    QString file_name = "User_Languages"+QString::number(lang_type)+".txt";
    QVector<QString> file = load_text_file(file_name);
    file.append(new_lang);
    prepare_lang_txt_for_dropdown(lang_type, false);
    write_text_file(file_name, file);
    ui->New_lang_input_field->clear();
    setup_to_add_new_lang(false, lang_type);
    setup_dropdown(false);

    ui->add_new_lang_label->setText(new_lang+" wurde hinzugefügt!");
}



⬇️ Hier wird das Userface daran angepasst, eine neue Sprache hinzufügen oder daran, wenn schon eien neue Sprache hinzugefügt wurde. Abhängig davon, ob add troe oder false ist. Je nach file Type,w ird ein anderer Dropdown angepasst.
void MainWindow::setup_to_add_new_lang(bool add, qint16 file_type){
    qInfo() << "das hinzufügen der neuen Sprache wird vorbereitet...";
    ui->back_tomenu_button3->setDisabled(add);
    ui->lang1_dropdown->setEnabled(!add);
    ui->lang2_dropdown->setEnabled(!add);

    if(file_type == 1){
        ui->lang1_dropdown->setCurrentIndex(-1);
    }else if(file_type == 2){
        ui->lang2_dropdown->setCurrentIndex(-1);
    }

    ui->New_lang_input_field->setVisible(add);
    ui->add_new_lang_label->setVisible(true);
    ui->New_lang_input_field->setFocus();
}



⬇️ Das Header label, welches die aktuellen Sprachen anzeigt, wird an die aktuell gewählten Sprachen angepasst.
bool MainWindow::set_languages_label(){
    qInfo() << "lang_label wird aktualisiert...";
    if(my_lang != "" && lang_to_learn != ""){
        if(first_time_user){
           new_user(1);
        }
        ui->header_languages->setVisible(true);
        ui->header_languages->setText(my_lang+" & "+lang_to_learn);
        return true;
    }
    return false;
}



⬇️ Hier wird beim Vokabeltest die Lösung angezeigt
qint16 MainWindow::show_solution(){
    qInfo() << "Lösung wird angezeigt...";
    QString vocabel_lang1 = ui->ask_voc_text_field_lang1->text();
    if(vocabel_lang1 != ""){
        qint16 index = findIndex(vocab_container_lang1, vocabel_lang1);
        QString solution = vocab_container_lang2[index];
        ui->ask_voc_text_field_lang2->setText(solution);
        ui->lang2_voc_label_add_text_field->setFocus();
        return index;
    }
    return 0;
}





⬇️  Hier wird eine neue Vokabel hinzugefügt. Dafür werden die aktuellen Vokabeln aus der Datenbank geladen, die in den Feldern zu den aktuellen Vokabeln hinzugefügt und dann in der Datenbank gespeichert.
void MainWindow::add_new_voc(){
    QString lang2_word = ui->lang2_voc_label_add_text_field->text();
    QString lang1_word = ui->lang1_voc_label_add_text_field->text();
    auto new_vocs = setup_vokabeltest();

    vocab_container_lang1 = new_vocs.first;
    vocab_container_lang2 = new_vocs.second;

vocab_container_lang1.append(lang1_word);
vocab_container_lang2.append(lang2_word);
write_text_file(my_lang+"_"+lang_to_learn+".txt", vocab_container_lang1);
write_text_file(lang_to_learn+"_"+my_lang+".txt", vocab_container_lang2);
qint16 voc_size = vocab_container_lang1.size();

 ui->lang2_voc_label_add_text_field->clear();
 ui->lang1_voc_label_add_text_field->clear();
 ui->voc_counter->display(voc_size);

 QString add_voc_message = "Vokabel gespeichert!";
 qint16 add_voc_time = 4000;
}




⬇️  Diese Funktion wird ausgeführt, wenn bei einer Vokabel im Vokabeltest auf “Bearbeiten” geklickt wird.
Diese Funktion ändert den Button Text auf “Bestätigen?”, lädt alle Vokabeln aus der Datenbank, entfernt die zu bearbeitende Vokabel und schreibt diesen bearbeiteten Vektor dann in die Datenbank. danach wird das zweite Sprachfeld auch noch für den Benutzer freigegeben.
void MainWindow::edit_voc1(QVector<QString> voc_lang2, QVector<QString> vocab_container_lang1){
    qInfo() << "Vokabel editieren start...";
    ui->edit_voc_button->setText("Bestätigen?");
    qint16 index = show_solution();

    vocab_container_lang1.removeAt(index);
    voc_lang2.removeAt(index);

    write_text_file(my_lang+"_"+lang_to_learn+".txt", vocab_container_lang1);
    write_text_file(lang_to_learn+"_"+my_lang+".txt", voc_lang2);

    ui->ask_voc_text_field_lang1->setReadOnly(false);
    ui->ask_voc_text_field_lang1->setFocus();
}



⬇️ Diese Funktion wird ausgeführt, nachdem die Vokabel editiert und dann bestätigt wurde.
Die neue Vokabeln werden zu den geladenen Vokabeln hinzugefügt und dann in die Datenbank geschrieben.
Danach wird das zweite Feld fürdie Vokabeln in der eigenen sprache wieder Readonly für den Benutzer + der Buttontext zurückgesetzt.
void MainWindow::edit_voc2(QVector<QString> vocab_container_lang2, QVector<QString> vocab_container_lang1){
    qInfo() << "Vokabel editieren ende...";
    QString vocabel_lang1_edit = ui->ask_voc_text_field_lang1->text();
    QString vocabel_lang2_edit = ui->ask_voc_text_field_lang2->text();

    vocab_container_lang1.append(vocabel_lang1_edit);
    vocab_container_lang2.append(vocabel_lang2_edit);

    write_text_file(my_lang+"_"+lang_to_learn+".txt", vocab_container_lang1);
    write_text_file(lang_to_learn+"_"+my_lang+".txt", vocab_container_lang2);

    ui->ask_voc_text_field_lang2->clear();
    ui->ask_voc_text_field_lang1->setReadOnly(true);
    ui->edit_voc_button->setText("Vokabel bearbeiten");
    setup_vokabeltest();
}


⬇️ Diese Funktion wird gestartet, wenn der Benutzer den Vokabeltest starten möchte. Dabei werden die Vokabeln aus der Datenbank geladen und dann in die Felder eine zufällige Vokabel eingetragen.
Falls in der Datenbank keine Vokabeln stehen, kommt ein Hinweis erst welche hinzuzufügen.
pair<QVector<QString>, QVector<QString> > MainWindow::setup_vokabeltest(){
    qInfo() << "Vokabeltest wird neu gestartet...";
    ui->voc_test_progressBar->setValue(0);
    vocab_container_lang1 = load_text_file(my_lang+"_"+lang_to_learn+".txt");
    vocab_container_lang2 = load_text_file(lang_to_learn+"_"+my_lang+".txt");
    qint16 voc_size = vocab_container_lang1.size();
    voc_size_saved = voc_size;

    if(vocab_container_lang1.size() > 0){
        ui->ask_voc_text_field_lang2->setFocus();
        ui->feedback_label_test->setText("Viel Glück!");
        qint16 random_index = get_random_index(vocab_container_lang1.size());
        QString start_voc = vocab_container_lang1[random_index];
        ui->ask_voc_text_field_lang1->setText(start_voc);
    }else{
        ui->feedback_label_test->setText("Füge erst Vokabeln hinzu!");
    }
    return make_pair(vocab_container_lang1, vocab_container_lang2);
}




⬇️ Diese Funktion wird ausgeführt, falls beim Vokabeltest, die richtige Vokabel eingetragen wurde. Dabei wird der Fortschrittscounter aktualisiert, in den temporären Daten die richtrige Vokabel in beiden Sprachen entfernt.
void MainWindow::vokabeltest_right(qint16 index_lang2, qint16 index_lang1, qint16 random_index, qreal voc_size){
    qInfo() << "Vokabeltest -> richtige Antwort...";
        qreal procent_vocs_solved = (qreal(voc_size_saved - (voc_size -1)) / qreal(voc_size_saved)) * 100;
        ui->voc_test_progressBar->setValue(procent_vocs_solved);
        vocab_container_lang1.removeAt(index_lang1);
        vocab_container_lang2.removeAt(index_lang2);

        if(voc_size > 1){
        QString next_voc = vocab_container_lang1[random_index];
        ui->ask_voc_text_field_lang1->setText(next_voc);

        ui->feedback_label_test->setText("Richtig!");
        }
    ui->ask_voc_text_field_lang2->clear();
}



⬇️ Sobald eine Vokabel im Vokabeltest eingegeben wurde, wird diese Funktion ausgeführt.
Diese Funktion gibt eienn boolean Wert zurück, dieser ist true, falls die Vokabel die eingegeben wird, die letzt in den Temporären Daten ist und richtig ist.
Falls beides der Fall ist, wird die Vokabel aus den Temporären Daten entfernt, der Fortschrittscounter aktualisiert und die Vokabelfelder geleert.
bool MainWindow::handle_last_voc(QString vocabel_lang1, QString vocabel_lang2, qint16 voc_size, qint16 index_lang1, qint16 index_lang2, qint16 random_index, QVector<QString> vocab_container_lang1, QVector<QString> vocab_container_lang2){
        if(voc_size == 1){
            qInfo() << "Letzte Vokabel wird gehandelt...";
            QString start_voc = vocab_container_lang1[random_index];
            ui->ask_voc_text_field_lang1->setText(start_voc);

            if(index_lang2 == index_lang1){
                ui->feedback_label_test->setText("Keine Vokabeln mehr übrig!");
                qreal procent_vocs_solved = (qreal(voc_size_saved - (voc_size -1)) / qreal(voc_size_saved)) * 100;
                ui->voc_test_progressBar->setValue(procent_vocs_solved);

                ui->ask_voc_text_field_lang1->clear();
                ui->ask_voc_text_field_lang2->clear();
                vocab_container_lang1.removeAt(0);
                vocab_container_lang2.removeAt(0);
                return true;
            }
            return false;
        }
    return false;
}


⬇️ Mit dieser Funktion wird der Index einer Vokabel in den Temporären Daten gesucht.
Dafür wird durch den Vektor gelooped und bei jedem Eintrag geschaut, ob der Eintrag gleich der Eingabe (“Word”) ist.
Dieser Index wird dann zurückgegeben.
qint16 MainWindow::findIndex(QVector<QString> Datei, QString Word){
    qInfo() << "index wird gesucht von: "+Word;
    qint16 index = -1;

    for (qint16 i = 0; i < Datei.size(); ++i) {
        QString Word2 = Datei[i];
        if (Word2 == Word) {
            index = i;
            break;
        }
    }
    return index;
}




⬇️ Mit dieser Funktion wird eine Random Zahl generiert zwischen 0 und dem größten Index aus den Vokabeln in den Temporären Daten in Sprache 1. Diese wird dann als Index für die nächste Vokabel benutzt.
qint16 MainWindow::get_random_index(qint16 voc_size){
    qInfo() << "random index wird geholt...";
    srand(time(0));
    int random_index = 0;
    if (voc_size > 1) {
        random_index = rand() % (vocab_container_lang1.size()-1);
    }
    return random_index;
}




⬇️ Diese Funktion wird jedes mal gestartet, wenn bei dem Vokabeltest eine EIngabe getätigt wird.
Dabei kann die Funktion mit dem boolean “neu” gestartet werden, dieser bedeutet, dass der Vokabeltest gestartet wird.
Falls keine Vokabel mehr übrig ist, wird die Funktion abgebrochen und das dem Benutzer mitgeteilt.
Falls der Vokabeltest  mit “neu” gestartet wird. werden die Vokabeln aus der Datenbank geladen und als temporäre Daten gesetzt. Sowie auch, dass dort dass mit “setup_Vokabeltest” eine zufällige Vokabel in das Fragefeld eingetragen wird,
Falls die Felder trotzdem leer sind, wird die Funktion an dieser Stelle auch abgebrochen, um einen error zu vermeiden.
Danach wird ein random Index generiert und es werden die Indizes der eingegebenen Vokabeln abgefragt. 
Danach wird die Funktion Last Voc Handler aufgerufen und bearbeitet. Wenn die eingegebene Vokabel die letzte ist, wird diese Funktion an diesem Punkt abgebrochen.
Falls die Vokabel 1 den gleichen Index wie den der eingegebenen hat, wird die Funktion “vokabeltest_right” gestartet. Dabei wird dann eine neue Vokabel eingesetzt.
Falls nicht, bekommt der Benutzer ein Feedback, dass die Vokabel falsch ist.


void MainWindow::vokabeltest(bool neu)
{
    qInfo() << "Vokabeltest...";
    qint16 voc_size = vocab_container_lang1.size();
    if (voc_size == 0 && !neu)
    {
        ui->feedback_label_test->setText("Keine Vokabeln mehr übrig!");
        return;
    }

    if (neu)
    {
        auto new_vocs = setup_vokabeltest();
        vocab_container_lang1 = new_vocs.first;
        vocab_container_lang2 = new_vocs.second;
        ui->voc_test_progressBar_label->setVisible(false);
        return;
    }


    QString vocabel_lang1 = ui->ask_voc_text_field_lang1->text();
    QString vocabel_lang2 = ui->ask_voc_text_field_lang2->text();

    if (vocabel_lang1.isEmpty())
    {
        return;
    }

    qint16 random_index = get_random_index(voc_size);
    qint16 index_lang1 = findIndex(vocab_container_lang1, vocabel_lang1);
    qint16 index_lang2 = findIndex(vocab_container_lang2, vocabel_lang2);

    bool last_voc_handler = handle_last_voc(vocabel_lang1, vocabel_lang2, voc_size, index_lang1, index_lang2, random_index, vocab_container_lang1, vocab_container_lang2);

    if (last_voc_handler)
    {
        return;
    }

    if (index_lang2 == index_lang1 && index_lang2 != -1)
    {
        vokabeltest_right(index_lang2, index_lang1, random_index, voc_size);
    }
    else
    {
        ui->feedback_label_test->setText("Leider falsch! :(");
    }
}



⬇️ Ab hier kommen keine normalen Funktionen mehr, sondern Funktionen die einen zugehörigen Trigger haben.
Zum Beispiel “on_BUTTON_NAME_clicked()” würde ausgelöst, wenn der Knopf mit dem Namen: “BUTTON_NAME” gedrückt wird.






⬇️ Dieser Knopf ist der, mit dem Vokabeln bearbeitet werden.
Wenn der Knopf gedrückt wird, werden die Vokabeln aus der Datenbank geladen und dann je nachdem, ob der Button zum ersten, oder zweiten mal gedrückt wird, die Funktion “edit_voc1” oder “edit_voc2” gestartet.
void MainWindow::on_edit_voc_button_clicked()
{
    vocab_container_lang1 = load_text_file(my_lang+"_"+lang_to_learn+".txt");
    vocab_container_lang2 = load_text_file(lang_to_learn+"_"+my_lang+".txt");
    QString vocabel_lang1_edit = ui->ask_voc_text_field_lang1->text();
    if(vocabel_lang1_edit != ""){
        if(ui->ask_voc_text_field_lang1->isReadOnly()){

            edit_voc1(vocab_container_lang2, vocab_container_lang1);

        }else{

            edit_voc2(vocab_container_lang2, vocab_container_lang1);

        }
    }
}




⬇️ Dieser Knopf ist der, der die Lösung beim Vokabeltest zeigt.
Wenn der Knopf gedrückt wird, wird “show_solution” gestartet, welche die Lösung anzeigt.
void MainWindow::on_show_solotion_button_clicked()
{
    show_solution();
}



⬇️ Dieser Knopf führt direkt zum Menü und triggert wenn nötig den nächsten Teil der Einweisung.
void MainWindow::on_back_to_menu_button1_clicked()
{
    if(first_time_user && vocab_container_lang1.size() > 0 ){
        new_user(3);
        message_to_user("", true, 0);
    }
    ui->stackedWidget->setCurrentWidget(ui->menu_page);

}



⬇️ Dieser Knopf führt direkt zum Menü und triggert wenn nötig den nächsten Teil der Einweisung. 
void MainWindow::on_back_to_menu_button2_clicked()
{
    if(first_time_user){
        new_user(-1);
    }
    ui->stackedWidget->setCurrentWidget(ui->menu_page);

}




⬇️ Dieser Knopf führt direkt zum Menü und triggert wenn nötig den nächsten Teil der Einweisung.
void MainWindow::on_back_tomenu_button3_clicked()
{
    bool label_set = set_languages_label();
    if(label_set && first_time_user){
        new_user(2);
    }
    ui->stackedWidget->setCurrentWidget(ui->menu_page);
}




⬇️ Dieser Knopf ist der, der im Menü zu dem Fenster Führt, in dem die Vokabeln hinzugefügt werden können.
Wenn der Knopf gedrückt wird, wird das entsprechende Fenster geöffnet, die Vokabeln werden neu geladen und die Seite wird je nach aktueller Sprache beschriftet.
void MainWindow::on_Menu_add_Vokabeln_clicked()
{
    if(first_time_user){
        message_to_user("", false, 0);
    }
    vocab_container_lang1 = load_text_file(my_lang+"_"+lang_to_learn+".txt");
    vocab_container_lang2 = load_text_file(lang_to_learn+"_"+my_lang+".txt");
    write_text_file(my_lang+"_"+lang_to_learn+".txt", vocab_container_lang1);
    write_text_file(lang_to_learn+"_"+my_lang+".txt", vocab_container_lang2);
    ui->lang1_voc_label_add->setText(my_lang);
    ui->lang2_voc_label_add->setText(lang_to_learn);
    ui->lang1_voc_label_add_text_field->setPlaceholderText(my_lang+"e Vokabel...");
    ui->lang2_voc_label_add_text_field->setPlaceholderText(lang_to_learn+"e Vokabel...");
    ui->stackedWidget->setCurrentWidget(ui->vokabel_add_page);
    qint16 voc_size = vocab_container_lang1.size();
    ui->voc_counter->display(voc_size);
}



⬇️ Dieser Knopf ist der, der im menü zum Vokabeltest führt.
Wenn der Knopf gedrückt wird, wird das Fenster für den Vokabeltest eingeblendet, die Vokabeln neu geladen und direkt eine zufällige Vokabel angezeigt und die Seite wird je nach aktueller Sprache beschriftet.
void MainWindow::on_Menu_Vokaltest_start_clicked()
{
    ui->ask_voc_text_field_lang1->setPlaceholderText(my_lang+"e Vokabel...");
    ui->ask_voc_text_field_lang2->setPlaceholderText(lang_to_learn+"e Vokabel...");
    ui->lang1_voc_label_test->setText(my_lang);
    ui->lang2_voc_label_test->setText(lang_to_learn);
    ui->stackedWidget->setCurrentWidget(ui->vokabel_test_page);
    vokabeltest(true);


}




⬇️ Das hier ist das erste Textfeld, zum hinzufügen von Vokabeln. Wenn hier eine Eingabe gesendet wird und beide Felder nicht leer sind, werden die Vokabeln zu der Datenbank hinzugefügt. Falls der User das zum ersten Mal macht, gibt es eine Feedback Meldung.
void MainWindow::on_lang1_voc_label_add_text_field_returnPressed()
{
    QString lang2_word = ui->lang2_voc_label_add_text_field->text();
    QString lang1_word = ui->lang1_voc_label_add_text_field->text();

    QString add_voc_message;
    qint16 add_voc_time;
        if( lang2_word != "" && lang1_word != ""){
            add_new_voc();
            ui->lang1_voc_label_add_text_field->setFocus();

         if(first_time_user && vocab_container_lang1.size() == 1){
             add_voc_message = "Du hast deine erste Vokabel gespeichert, gut gemacht! Fuege ruhig noch weitere hinzu.";
             add_voc_time = 15000;
         }
         message_to_user(add_voc_message, false, add_voc_time);
        }   else if(lang1_word != ""){
            ui->lang2_voc_label_add_text_field->setFocus();
        }
}




⬇️ Das hier ist das zweite Textfeld, zum hinzufügen von Vokabeln. Wenn hier eine Eingabe gesendet wird und beide Felder nicht leer sind, werden die Vokabeln zu der Datenbank hinzugefügt. Falls der User das zum ersten Mal macht, gibt es eine Feedback Meldung.
void MainWindow::on_lang2_voc_label_add_text_field_returnPressed()
{
    QString lang1_word = ui->lang1_voc_label_add_text_field->text();
    QString lang2_word = ui->lang2_voc_label_add_text_field->text();

    QString add_voc_message;
    qint16 add_voc_time;

        if( lang1_word != "" && lang2_word != ""){
            add_new_voc();
            ui->lang1_voc_label_add_text_field->setFocus();

        if(first_time_user && vocab_container_lang1.size() == 1){
            add_voc_message = "Du hast deine erste Vokabel gespeichert, gut gemacht! Fuege ruhig noch weitere hinzu.";
            add_voc_time = 10000;
        }
        message_to_user(add_voc_message, false, add_voc_time);

        }   else if(lang2_word != ""){
            ui->lang1_voc_label_add_text_field->setFocus();
        }

}




⬇️ Falls beim Vokabeltest eine Eingabe ins Eingabefeld gesendet wird, wird geschaut, ob das andere Feld Readonly ist. Falls das nämlich nicht der Fall ist, wird gerade eine vokabel bearbeitet und es wird nur das andere Feld fokussiert. Falls doch, dann wird geprüft, ob die Vokabel richtig ist und der Vokabeltest fortgesetzt.
void MainWindow::on_ask_voc_text_field_lang2_returnPressed()
{
    if(ui->ask_voc_text_field_lang1->isReadOnly()){
        vokabeltest(false);
    }else{
        ui->ask_voc_text_field_lang1->setFocus();
    }
}




⬇️ Dieser Knopf ist der, der den Vokabeltest neustartet.
Wenn der Knopf gedrückt wird, werden die Vokabeln neu geladen.
void MainWindow::on_restart_voc_test_clicked()
{
    vokabeltest(true);
}




⬇️ Falls beim Vokabeltest in diesem Feld gelinksklickt wird, wird das andere Feld fokussiert.
void MainWindow::on_ask_voc_text_field_lang1_returnPressed()
{
    ui->ask_voc_text_field_lang2->setFocus();
}



⬇️ Das hier ist ein Event, dass ausgelöst wird, wenn mit der Maus gelinksklickt wird. Dabei wird m_dragging true, wodurch das Fenster dem Curser folgt.
void MainWindow::mousePressEvent(QMouseEvent* event)
{
    if (event->button() == Qt::LeftButton) {
        m_dragging = true;
        m_lastPos = QPoint(event->globalPosition().x(), event->globalPosition().y());
    }
}







⬇️ Hier wird festgelegt, dass wenn m_dragginbg true ist, das Fenster immer dem Curser folgt.
void MainWindow::mouseMoveEvent(QMouseEvent* event)
{
    if (m_dragging) {
        int xDiff = event->globalPosition().x() * devicePixelRatio() - m_lastPos.x();
        int yDiff = event->globalPosition().y() * devicePixelRatio() - m_lastPos.y();
        move(this->x() + xDiff, this->y() + yDiff);
        m_lastPos = QPoint(event->globalPosition().x(), event->globalPosition().y());
    }
}






⬇️ Das hier ist ein Event, dass ausgelöst wird, wenn mit der Maus gelinksklickt wurde und der Knopf losgelöst wird. Dabei wird m_dragging false, wodurch das Fenster dem Curser nicht mehr folgt.
void MainWindow::mouseReleaseEvent(QMouseEvent* event)
{
    if (event->button() == Qt::LeftButton) {
        m_dragging = false;
    }
}




⬇️ Dieser Knopf ist der, der im Haumptmenü zu den Settings führt.
Wenn der Knopf gedrückt wird, wird das Einestellungen Fenster eingeblendet
void MainWindow::on_Menu_settings_clicked()
{
    ui->stackedWidget->setCurrentWidget(ui->settings_page);
    setup_dropdown(true);

    if(first_time_user){
        message_to_user("", false, 0);
    }
}



⬇️ Hier ist das Input Feld, dass sichtbar wird, wenn eine Neue Sprache hinzugefügt werden soll. Sobald in diesem Feld eine EIngabe gesendet wird, wird geschaut, zu welchem Sprachtyp die neue Sprache hinzugefügt werden soll und je nachdem, wird die Sprache hinzugefügt.
void MainWindow::on_New_lang_input_field_returnPressed()
{
    QString new_lang = ui->New_lang_input_field->text();
    new_language(which_file_to_edit, new_lang);
    if(which_file_to_edit == 1){
        ui->lang1_dropdown->setCurrentText(new_lang);
    }else if(which_file_to_edit == 2){
        ui->lang2_dropdown->setCurrentText(new_lang);
    }
}





⬇️ Falls in den Einstellungen im Dropdown 2 ein anderes Element ausgewählt wird, wird geprüft, ob eine “Neue Sprache” hinzugefügt werden soll, wenn ja, dann wird das Eingabefeld dafür sichtbar und wenn nicht, werden einfach nur die Sprachen aktualisiert. 
void MainWindow::on_lang2_dropdown_currentIndexChanged(int index)
{
    QString lang2 = ui->lang2_dropdown->currentText();
        if(lang2 == "Neue Sprache"){
            ui->New_lang_input_field->setPlaceholderText("Die Sprache die du lernen willst hinzufügen...");
           setup_to_add_new_lang(true, 2);
           which_file_to_edit = 2;
        }else if(index>-1){
            qInfo() << "last_languages.txt soll geschrieben werden?";
            lang_to_learn = lang2;
            write_text_file("last_languages.txt", {my_lang,lang_to_learn});
            set_languages_label();
        }
}





⬇️ Falls in den Einstellungen im Dropdown 1 ein anderes Element ausgewählt wird, wird geprüft, ob eine “Neue Sprache” hinzugefügt werden soll, wenn ja, dann wird das Eingabefeld dafür sichtbar und wenn nicht, werden einfach nur die Sprachen aktualisiert.
void MainWindow::on_lang1_dropdown_currentIndexChanged(int index)
{
    QString lang1 = ui->lang1_dropdown->currentText();
    if(lang1 == "Neue Sprache"){
       ui->New_lang_input_field->setPlaceholderText("Deine Sprache hinzufügen...");
       setup_to_add_new_lang(true, 1);
       which_file_to_edit = 1;
    }else if(index>-1){
        my_lang = lang1;
        qInfo() << "last_languages.txt soll geschrieben werden?";
        write_text_file("last_languages.txt", {my_lang,lang_to_learn});
        set_languages_label();
    }
}




⬇️ Falls beim Vokabeltest die Fortschrittsanzeige bei 100% liegt, also der Test fertig ist, dann wird das Text Label “Gut gemacht!” eingeblendet
void MainWindow::on_voc_test_progressBar_valueChanged(int value)
{
    if(value == 100){
        ui->voc_test_progressBar_label->setVisible(true);

        message_to_user("Gut gemacht!!", true, 6500);
    }
}





