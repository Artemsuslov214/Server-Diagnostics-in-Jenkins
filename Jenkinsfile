pipeline {

    agent any

    parameters {
        string(
            name: 'SERVER',
            defaultValue: '192.168.56.10',
            description: 'Target server'
        )

        string(
            name: 'SSH_USER',
            defaultValue: 'artem',
            description: 'SSH user on target server'
        )

        string(
            name: 'SERVICE',
            defaultValue: 'nginx',
            description: 'systemd service name, without .service'
        )

        string(
            name: 'PORT',
            defaultValue: '80',
            description: 'Expected TCP port'
        )
    }

    environment {
        REPORT = 'diagnostic-report.txt'
    }

    stages {

        stage('Initialize') {
            steps {
                sh '''
                    rm -f "${REPORT}"

                    echo "========================================" | tee -a "${REPORT}"
                    echo "SERVER DIAGNOSTIC REPORT" | tee -a "${REPORT}"
                    echo "========================================" | tee -a "${REPORT}"
                    echo "" | tee -a "${REPORT}"
                    echo "" | tee -a "${REPORT}"

                    echo "Host    : ${SERVER}" | tee -a "${REPORT}"
                    echo "Port    : ${PORT}" | tee -a "${REPORT}"
                    echo "Service : ${SERVICE}" | tee -a "${REPORT}"
                    echo "SSH User: ${SSH_USER}" | tee -a "${REPORT}"
                    echo "Date    : $(date -u '+%Y-%m-%d %H:%M:%S UTC')" | tee -a "${REPORT}"

                    echo "" | tee -a "${REPORT}"
                    echo "" | tee -a "${REPORT}"
                '''
            }
        }


        stage('Network Diagnostics') {
            steps {
                sh '''
                    set +e

                    echo "========================================" | tee -a "${REPORT}"
                    echo "NETWORK DIAGNOSTICS" | tee -a "${REPORT}"
                    echo "========================================" | tee -a "${REPORT}"
                    echo "" | tee -a "${REPORT}"
                    echo "" | tee -a "${REPORT}"

                    echo "Source: Jenkins Agent" | tee -a "${REPORT}"
                    echo "Target: ${SERVER}" | tee -a "${REPORT}"
                    echo "" | tee -a "${REPORT}"
                    echo "" | tee -a "${REPORT}"


                    # ========================================
                    # PING
                    # ========================================

                    echo "========== PING ==========" | tee -a "${REPORT}"

                    if command -v ping >/dev/null 2>&1; then

                        PING_OUTPUT=$(ping -c 4 -W 2 "${SERVER}" 2>&1)
                        PING_RESULT=$?

                        echo "${PING_OUTPUT}" | tee -a "${REPORT}"

                        if [ "${PING_RESULT}" -eq 0 ]; then
                            echo "PING RESULT: OK" | tee -a "${REPORT}"
                        else
                            echo "PING RESULT: FAILED" | tee -a "${REPORT}"
                        fi

                    else

                        echo "ping is not installed on Jenkins agent" | tee -a "${REPORT}"
                        echo "PING RESULT: UNKNOWN" | tee -a "${REPORT}"

                    fi

                    echo "" | tee -a "${REPORT}"
                    echo "" | tee -a "${REPORT}"


                    # ========================================
                    # TCP PORT CHECK
                    # ========================================

                    echo "========== TCP PORT CHECK ==========" | tee -a "${REPORT}"

                    if command -v nc >/dev/null 2>&1; then

                        NC_OUTPUT=$(nc -zv -w 5 "${SERVER}" "${PORT}" 2>&1)
                        NC_RESULT=$?

                        echo "${NC_OUTPUT}" | tee -a "${REPORT}"

                        if [ "${NC_RESULT}" -eq 0 ]; then
                            echo "PORT RESULT: OPEN" | tee -a "${REPORT}"
                        else
                            echo "PORT RESULT: CLOSED OR FILTERED" | tee -a "${REPORT}"
                        fi

                    elif command -v telnet >/dev/null 2>&1; then

                        TELNET_OUTPUT=$(timeout 5 telnet "${SERVER}" "${PORT}" 2>&1)
                        TELNET_RESULT=$?

                        echo "${TELNET_OUTPUT}" | tee -a "${REPORT}"

                        if [ "${TELNET_RESULT}" -eq 0 ]; then
                            echo "PORT RESULT: OPEN" | tee -a "${REPORT}"
                        else
                            echo "PORT RESULT: CLOSED OR FILTERED" | tee -a "${REPORT}"
                        fi

                    else

                        echo "Neither nc nor telnet is installed on Jenkins agent" | tee -a "${REPORT}"
                        echo "PORT RESULT: UNKNOWN" | tee -a "${REPORT}"

                    fi

                    echo "" | tee -a "${REPORT}"
                    echo "" | tee -a "${REPORT}"


                    # ========================================
                    # HTTP CHECK FROM JENKINS AGENT
                    # ========================================

                    echo "========== HTTP CHECK ==========" | tee -a "${REPORT}"

                    if command -v curl >/dev/null 2>&1; then

                        HTTP_OUTPUT=$(curl -I -m 10 "http://${SERVER}:${PORT}" 2>&1)
                        HTTP_RESULT=$?

                        echo "${HTTP_OUTPUT}" | tee -a "${REPORT}"

                        if [ "${HTTP_RESULT}" -eq 0 ]; then
                            echo "HTTP RESULT: REACHABLE" | tee -a "${REPORT}"
                        else
                            echo "HTTP RESULT: FAILED" | tee -a "${REPORT}"
                        fi

                    else

                        echo "curl is not installed on Jenkins agent" | tee -a "${REPORT}"
                        echo "HTTP RESULT: UNKNOWN" | tee -a "${REPORT}"

                    fi

                    echo "" | tee -a "${REPORT}"
                    echo "" | tee -a "${REPORT}"


                    # ========================================
                    # TRACEROUTE
                    # ========================================

                    echo "========== TRACEROUTE ==========" | tee -a "${REPORT}"

                    if command -v traceroute >/dev/null 2>&1; then

                        TRACEROUTE_OUTPUT=$(timeout 10 traceroute -n -w 1 -q 1 -m 10 "${SERVER}" 2>&1)
                        TRACEROUTE_RC=$?

                        if [ "${TRACEROUTE_RC}" -eq 0 ]; then
                            echo "${TRACEROUTE_OUTPUT}" | tee -a "${REPORT}"
                            echo "TRACEROUTE RESULT: OK" | tee -a "${REPORT}"

                        elif [ "${TRACEROUTE_RC}" -eq 124 ]; then
                            echo "Traceroute timed out after 10 seconds" | tee -a "${REPORT}"
                            echo "TRACEROUTE RESULT: TIMEOUT" | tee -a "${REPORT}"

                        else
                            echo "${TRACEROUTE_OUTPUT}" | tee -a "${REPORT}"
                            echo "TRACEROUTE RESULT: FAILED" | tee -a "${REPORT}"
                        fi

                    elif command -v tracepath >/dev/null 2>&1; then

                        TRACEPATH_OUTPUT=$(timeout 10 tracepath -n "${SERVER}" 2>&1)
                        TRACEPATH_RC=$?

                        if [ "${TRACEPATH_RC}" -eq 0 ]; then
                            echo "${TRACEPATH_OUTPUT}" | tee -a "${REPORT}"
                            echo "TRACEPATH RESULT: OK" | tee -a "${REPORT}"

                        elif [ "${TRACEPATH_RC}" -eq 124 ]; then
                            echo "Tracepath timed out after 10 seconds" | tee -a "${REPORT}"
                            echo "TRACEPATH RESULT: TIMEOUT" | tee -a "${REPORT}"

                        else
                            echo "${TRACEPATH_OUTPUT}" | tee -a "${REPORT}"
                            echo "TRACEPATH RESULT: FAILED" | tee -a "${REPORT}"
                        fi

                    else

                        echo "traceroute/tracepath is not installed on Jenkins agent" | tee -a "${REPORT}"
                        echo "TRACEROUTE RESULT: NOT AVAILABLE" | tee -a "${REPORT}"

                    fi

                    echo "" | tee -a "${REPORT}"
                    echo "" | tee -a "${REPORT}"


                    # ========================================
                    # MTR
                    # ========================================

                    echo "========== MTR ==========" | tee -a "${REPORT}"

                    if command -v mtr >/dev/null 2>&1; then

                        MTR_OUTPUT=$(timeout 10 mtr -n -r -c 5 "${SERVER}" 2>&1)
                        MTR_RC=$?

                        if [ "${MTR_RC}" -eq 0 ]; then

                            echo "${MTR_OUTPUT}" | tee -a "${REPORT}"
                            echo "MTR RESULT: OK" | tee -a "${REPORT}"

                        elif [ "${MTR_RC}" -eq 124 ]; then

                            echo "${MTR_OUTPUT}" | tee -a "${REPORT}"
                            echo "MTR RESULT: TIMEOUT" | tee -a "${REPORT}"

                        else

                            echo "${MTR_OUTPUT}" | tee -a "${REPORT}"
                            echo "MTR RESULT: FAILED" | tee -a "${REPORT}"

                        fi

                    else

                        echo "mtr is not installed on Jenkins agent" | tee -a "${REPORT}"
                        echo "MTR RESULT: NOT AVAILABLE" | tee -a "${REPORT}"

                    fi

                    echo "" | tee -a "${REPORT}"
                    echo "" | tee -a "${REPORT}"

                    set -e
                '''
            }
        }


        stage('Server Diagnostics') {
            steps {
                sh '''
                    set +e

                    echo "========================================" | tee -a "${REPORT}"
                    echo "SERVER DIAGNOSTICS" | tee -a "${REPORT}"
                    echo "========================================" | tee -a "${REPORT}"
                    echo "" | tee -a "${REPORT}"
                    echo "" | tee -a "${REPORT}"


                    # ========================================
                    # SSH
                    # ========================================

                    ssh \
                        -o BatchMode=yes \
                        -o ConnectTimeout=10 \
                        -o StrictHostKeyChecking=accept-new \
                        "${SSH_USER}@${SERVER}" \
                        "SERVICE='${SERVICE}' PORT='${PORT}' bash -s" \
                        <<'REMOTE' >> "${REPORT}" 2>&1

set +e

echo "========== HOST =========="
hostname


echo ""
echo "========== UPTIME =========="
uptime


echo ""
echo ""
echo "========== SERVICE STATE =========="

SERVICE_ACTIVE=\$(systemctl is-active "\${SERVICE}.service" 2>/dev/null)
SERVICE_ENABLED=\$(systemctl is-enabled "\${SERVICE}.service" 2>/dev/null)

echo "\${SERVICE_ACTIVE}"
echo "\${SERVICE_ENABLED}"

if [ "\${SERVICE_ACTIVE}" = "active" ]; then
    echo "SERVICE RESULT: OK"
else
    echo "SERVICE RESULT: NOT RUNNING"
fi


echo ""
echo ""
echo "========== SERVICE STATUS =========="

systemctl status "\${SERVICE}.service" --no-pager -l


echo ""
echo ""
echo "========== SERVICE LOGS =========="

journalctl \
    -u "\${SERVICE}.service" \
    -n 50 \
    --no-pager \
    2>&1


echo ""
echo ""
echo "========== PROCESS =========="

MAIN_PID=\$(systemctl show \
    "\${SERVICE}.service" \
    --property=MainPID \
    --value \
    2>/dev/null)

if [ -n "\${MAIN_PID}" ] && [ "\${MAIN_PID}" != "0" ]; then

    ps -fp "\${MAIN_PID}"

    echo ""
    echo "Process children:"

    ps --ppid "\${MAIN_PID}" -f 2>/dev/null

else

    echo "Main process not found"

fi


echo ""
echo ""
echo "========== TOP PROCESSES BY CPU =========="

ps aux --sort=-%cpu | head -n 6


echo ""
echo ""
echo "========== LISTENING PORT =========="

if command -v ss >/dev/null 2>&1; then

    SS_OUTPUT=\$(ss -lntup 2>/dev/null | grep -E ":\${PORT}([[:space:]]|$)")

    if [ -n "\${SS_OUTPUT}" ]; then

        echo "\${SS_OUTPUT}"
        echo "PORT RESULT: LISTENING"

    else

        echo "No process is listening on port \${PORT}"
        echo "PORT RESULT: NOT LISTENING"

    fi

else

    echo "ss is not installed"

fi


echo ""
echo ""
echo "========== MEMORY =========="

free -h


echo ""
echo ""
echo "========== DISK =========="

df -h


echo ""
echo ""
echo "========== OOM =========="

if command -v dmesg >/dev/null 2>&1; then

    OOM_OUTPUT=\$(dmesg 2>&1 | grep -iE "out of memory|oom|killed process")

    if [ -n "\${OOM_OUTPUT}" ]; then

        echo "\${OOM_OUTPUT}"
        echo ""
        echo "OOM RESULT: DETECTED"

    else

        echo "No OOM messages found"
        echo "OOM RESULT: NOT DETECTED"

    fi

else

    echo "dmesg is not available"
    echo "OOM RESULT: UNKNOWN"

fi


echo ""
echo ""
echo "========== LOCAL HTTP CHECK =========="

if command -v curl >/dev/null 2>&1; then

    LOCAL_HTTP_OUTPUT=\$(curl -I -m 10 "http://127.0.0.1:\${PORT}" 2>&1)
    LOCAL_HTTP_RESULT=\$?

    echo "\${LOCAL_HTTP_OUTPUT}"

    if [ "\${LOCAL_HTTP_RESULT}" -eq 0 ]; then
        echo "LOCAL HTTP RESULT: REACHABLE"
    else
        echo "LOCAL HTTP RESULT: FAILED"
    fi

else

    echo "curl is not installed"
    echo "LOCAL HTTP RESULT: UNKNOWN"

fi


echo ""
echo ""
echo "SSH RESULT: OK"

REMOTE

                    SSH_RESULT=$?

                    if [ "${SSH_RESULT}" -ne 0 ]; then

                        echo "" | tee -a "${REPORT}"
                        echo "" | tee -a "${REPORT}"
                        echo "SSH RESULT: FAILED" | tee -a "${REPORT}"
                        echo "Unable to execute diagnostics on remote server." | tee -a "${REPORT}"
                        echo "SSH exit code: ${SSH_RESULT}" | tee -a "${REPORT}"

                    fi

                    set -e
                '''
            }
        }


        stage('Create Summary') {
            steps {
                sh '''
                    set +e

                    echo "" >> "${REPORT}"
                    echo "" >> "${REPORT}"
                    echo "" >> "${REPORT}"
                    echo "========================================" >> "${REPORT}"
                    echo "DIAGNOSTIC SUMMARY" >> "${REPORT}"
                    echo "========================================" >> "${REPORT}"
                    echo "" >> "${REPORT}"
                    echo "" >> "${REPORT}"

                    echo "Host    : ${SERVER}" >> "${REPORT}"
                    echo "Port    : ${PORT}" >> "${REPORT}"
                    echo "Service : ${SERVICE}" >> "${REPORT}"
                    echo "Date    : $(date -u '+%Y-%m-%d %H:%M:%S UTC')" >> "${REPORT}"
                    echo "" >> "${REPORT}"
                    echo "" >> "${REPORT}"


                    # ========================================
                    # NETWORK
                    # ========================================

                    echo "NETWORK" >> "${REPORT}"

                    if grep -q "PING RESULT: OK" "${REPORT}"; then
                        echo "  Ping          : OK" >> "${REPORT}"
                    elif grep -q "PING RESULT: FAILED" "${REPORT}"; then
                        echo "  Ping          : FAILED" >> "${REPORT}"
                    else
                        echo "  Ping          : UNKNOWN" >> "${REPORT}"
                    fi


                    if grep -q "PORT RESULT: OPEN" "${REPORT}"; then
                        echo "  TCP ${PORT}       : OPEN" >> "${REPORT}"
                    elif grep -q "PORT RESULT: CLOSED OR FILTERED" "${REPORT}"; then
                        echo "  TCP ${PORT}       : CLOSED/FILTERED" >> "${REPORT}"
                    else
                        echo "  TCP ${PORT}       : UNKNOWN" >> "${REPORT}"
                    fi


                    if grep -q "HTTP RESULT: REACHABLE" "${REPORT}"; then
                        echo "  HTTP          : REACHABLE" >> "${REPORT}"
                    elif grep -q "HTTP RESULT: FAILED" "${REPORT}"; then
                        echo "  HTTP          : FAILED" >> "${REPORT}"
                    else
                        echo "  HTTP          : UNKNOWN" >> "${REPORT}"
                    fi


                    if grep -q "MTR RESULT: OK" "${REPORT}"; then
                        echo "  MTR           : OK" >> "${REPORT}"
                    elif grep -q "MTR RESULT: FAILED" "${REPORT}"; then
                        echo "  MTR           : FAILED" >> "${REPORT}"
                    else
                        echo "  MTR           : UNKNOWN" >> "${REPORT}"
                    fi


                    echo "" >> "${REPORT}"
                    echo "" >> "${REPORT}"


                    # ========================================
                    # SERVER
                    # ========================================

                    echo "SERVER" >> "${REPORT}"


                    if grep -q "SSH RESULT: OK" "${REPORT}"; then
                        echo "  SSH           : OK" >> "${REPORT}"
                    else
                        echo "  SSH           : FAILED" >> "${REPORT}"
                    fi


                    if grep -q "SERVICE RESULT: OK" "${REPORT}"; then
                        echo "  Service       : RUNNING" >> "${REPORT}"
                    elif grep -q "SERVICE RESULT: NOT RUNNING" "${REPORT}"; then
                        echo "  Service       : NOT RUNNING" >> "${REPORT}"
                    else
                        echo "  Service       : UNKNOWN" >> "${REPORT}"
                    fi


                    if grep -A3 "========== SERVICE STATE ==========" "${REPORT}" | \
                        grep -q "^enabled$"; then

                        echo "  Enabled       : ENABLED" >> "${REPORT}"

                    else

                        echo "  Enabled       : UNKNOWN/DISABLED" >> "${REPORT}"

                    fi


                    if grep -q "PORT RESULT: LISTENING" "${REPORT}"; then
                        echo "  Listening     : YES" >> "${REPORT}"
                    elif grep -q "PORT RESULT: NOT LISTENING" "${REPORT}"; then
                        echo "  Listening     : NO" >> "${REPORT}"
                    else
                        echo "  Listening     : UNKNOWN" >> "${REPORT}"
                    fi


                    echo "" >> "${REPORT}"
                    echo "" >> "${REPORT}"


                    # ========================================
                    # RESOURCES
                    # ========================================

                    echo "RESOURCES" >> "${REPORT}"


                    if grep -q "OOM RESULT: DETECTED" "${REPORT}"; then
                        echo "  OOM           : DETECTED" >> "${REPORT}"
                    elif grep -q "OOM RESULT: NOT DETECTED" "${REPORT}"; then
                        echo "  OOM           : NOT DETECTED" >> "${REPORT}"
                    else
                        echo "  OOM           : UNKNOWN" >> "${REPORT}"
                    fi


                    echo "  Root Disk     : See DISK section" >> "${REPORT}"
                    echo "  Memory        : See MEMORY section" >> "${REPORT}"


                    echo "" >> "${REPORT}"
                    echo "" >> "${REPORT}"


                    # ========================================
                    # RESULT
                    # ========================================

                    echo "RESULT" >> "${REPORT}"


                    if grep -q "SERVICE RESULT: OK" "${REPORT}" && \
                       grep -q "PORT RESULT: LISTENING" "${REPORT}"; then

                        echo "  SERVICE IS RUNNING AND PORT IS LISTENING" >> "${REPORT}"

                    elif grep -q "SERVICE RESULT: OK" "${REPORT}"; then

                        echo "  SERVICE IS RUNNING BUT PORT STATUS REQUIRES ATTENTION" >> "${REPORT}"

                    elif grep -q "SERVICE RESULT: NOT RUNNING" "${REPORT}"; then

                        echo "  SERVICE IS NOT RUNNING" >> "${REPORT}"

                    else

                        echo "  UNABLE TO DETERMINE SERVICE STATE" >> "${REPORT}"

                    fi


                    echo "" >> "${REPORT}"
                    echo "" >> "${REPORT}"
                    echo "========================================" >> "${REPORT}"
                    echo "" >> "${REPORT}"
                    echo "" >> "${REPORT}"

                    set -e
                '''
            }
        }


        stage('Archive Report') {
            steps {
                archiveArtifacts(
                    artifacts: 'diagnostic-report.txt',
                    fingerprint: true,
                    allowEmptyArchive: false
                )

                echo "Diagnostic report archived: diagnostic-report.txt"
            }
        }
    }


    post {
        always {
            echo "Server diagnostic job finished."
        }
    }
}
