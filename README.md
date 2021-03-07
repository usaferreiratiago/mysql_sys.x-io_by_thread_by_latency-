# mysql_sys.x-io_by_thread_by_latency-

SELECT 
    IF((`performance_schema`.`threads`.`PROCESSLIST_ID` IS NULL),
        SUBSTRING_INDEX(`performance_schema`.`threads`.`NAME`,
                '/',
                -(1)),
        CONCAT(`performance_schema`.`threads`.`PROCESSLIST_USER`,
                '@',
                CONVERT( `performance_schema`.`threads`.`PROCESSLIST_HOST` USING UTF8MB4))) AS `user`,
    SUM(`performance_schema`.`events_waits_summary_by_thread_by_event_name`.`COUNT_STAR`) AS `total`,
    SUM(`performance_schema`.`events_waits_summary_by_thread_by_event_name`.`SUM_TIMER_WAIT`) AS `total_latency`,
    MIN(`performance_schema`.`events_waits_summary_by_thread_by_event_name`.`MIN_TIMER_WAIT`) AS `min_latency`,
    AVG(`performance_schema`.`events_waits_summary_by_thread_by_event_name`.`AVG_TIMER_WAIT`) AS `avg_latency`,
    MAX(`performance_schema`.`events_waits_summary_by_thread_by_event_name`.`MAX_TIMER_WAIT`) AS `max_latency`,
    `performance_schema`.`events_waits_summary_by_thread_by_event_name`.`THREAD_ID` AS `thread_id`,
    `performance_schema`.`threads`.`PROCESSLIST_ID` AS `processlist_id`
FROM
    (`performance_schema`.`events_waits_summary_by_thread_by_event_name`
    LEFT JOIN `performance_schema`.`threads` ON ((`performance_schema`.`events_waits_summary_by_thread_by_event_name`.`THREAD_ID` = `performance_schema`.`threads`.`THREAD_ID`)))
WHERE
    ((`performance_schema`.`events_waits_summary_by_thread_by_event_name`.`EVENT_NAME` LIKE 'wait/io/file/%')
        AND (`performance_schema`.`events_waits_summary_by_thread_by_event_name`.`SUM_TIMER_WAIT` > 0))
GROUP BY `performance_schema`.`events_waits_summary_by_thread_by_event_name`.`THREAD_ID` , `performance_schema`.`threads`.`PROCESSLIST_ID` , `user`
ORDER BY SUM(`performance_schema`.`events_waits_summary_by_thread_by_event_name`.`SUM_TIMER_WAIT`) DESC
